# Evaluation Report — Agentic Engineering Study Assistant (Track 1: n8n)

**Method:** 15 questions in 3 buckets (5 straightforward, 5 ambiguous/multi-document, 5 unanswerable traps), asked one at a time in the n8n chat panel. For each question I record what was retrieved (the cited file names), whether the answer was faithful to the corpus, and the failure mode if any.

**Scoring rubric**
- **Retrieval (0–2):** 2 = answer content shows the right docs were retrieved · 1 = partially relevant · 0 = wrong/no retrieval
- **Faithful (Y/N):** every claim in the answer is supported by the corpus (for traps: a refusal counts as faithful; any invented answer = N)
- **Citation (Y/N):** answer lists the correct, real source file name(s)
- **Failure mode (if failed):** `retrieval-miss` · `generation-drift` · `over-refusal` · `hallucination` · `metadata-loss` (correct retrieval, but source names not preserved/fabricated)

---

## Bucket A — Straightforward (single-document answers)

| # | Question | Expected source(s) | Retrieval | Faithful | Citation | Failure mode | Notes |
|---|---|---|---|---|---|---|---|
| 1 | According to Anthropic, what is the difference between a workflow and an agent? | blog-building-effective-agents.md | 2 | Y | N | metadata-loss | Correct distinction (predefined code paths vs dynamic self-direction); sources shown as UUID filenames |
| 2 | What are the three core building blocks an MCP server can expose to clients? | mcp-server-concepts.md | 2 | Y | ~Y | — | Tools/Resources/Prompts with correct definitions; cited "server-concepts.md" (close but not exact filename) |
| 3 | What is the ReAct pattern, and what two capabilities does it interleave? | paper-react-reasoning-acting-llms.md | 2 | Y | N | metadata-loss | Correct (reasoning + acting), though terse; UUID citations |
| 4 | What is a checkpointer in LangGraph and what does it enable? | langgraph-persistence.md | 2 | Y | N | metadata-loss | Excellent: BaseCheckpointSaver, snapshots, HITL, memory, time travel, fault tolerance — all grounded; UUID citations |
| 5 | What does Anthropic recommend for writing good tool descriptions? | blog-writing-tools-for-agents.md, anthropic-define-tools.md | 2 | Y | N | metadata-loss | Grounded (clear plaintext descriptions, when/how to use); UUID citations |

## Bucket B — Ambiguous or multi-document

| # | Question | Expected source(s) | Retrieval | Faithful | Citation | Failure mode | Notes |
|---|---|---|---|---|---|---|---|
| 6 | How do agents remember things? | anthropic-memory-tool.md, langgraph-add-memory.md | 1 | Y | N | metadata-loss + partial retrieval | Accurate description of Anthropic's memory tool, but the LangGraph memory perspective was missed and no clarifying question was asked (prompt rule 4 not triggered). Cited "langgraph.md" — **a file that does not exist**: fabricated citation |
| 7 | What's the best way to manage context in a long-running agent? | blog-effective-context-engineering-for-ai-agents.md, anthropic-context-windows/editing | 2 | Y | N | metadata-loss | Content is squarely from the context-engineering blog (compaction, structured notes, sub-agents) but cited blog-building-effective-agents.md + mcp-architecture.md — plausible-looking **wrong** files |
| 8 | How should I debug my MCP integration? | mcp-debugging.md, mcp-inspector.md | 2 | Y | N | metadata-loss | Faithful to mcp-debugging.md structure (logs, Inspector, support channels); UUID citations |
| 9 | What security risks should I watch out for when building MCP servers? | mcp-security_best_practices.md, mcp-authorization.md | — | — | — | — | **Run and fill in** |
| 10 | When should I use multiple agents instead of a single agent? | blog-multi-agent-research-system.md, blog-building-effective-agents.md | 2 | Y | N | metadata-loss | Correct framing (open-ended problems, unpredictable subtasks, orchestrator-workers); cited "langgraph-docs.md" — **does not exist**: fabricated citation |

## Bucket C — Unanswerable traps (correct behavior = refusal)

| # | Question | Why it's a trap | Refused? | Faithful | Failure mode | Notes |
|---|---|---|---|---|---|---|
| 11 | What does the corpus say about Kubernetes autoscaling? | Out of domain entirely | Y | Y | — | Clean refusal + sensible redirect |
| 12 | What does OpenAI's Swarm framework recommend for agent handoffs? | Plausible topic, not in corpus | Y | Y | — | Clean refusal |
| 13 | What is the per-token price of Claude Opus? | Pricing nowhere in corpus | Y | Y | — | Refused, though the suggestion ("Pricing section of the models overview table") implies knowledge of a doc structure it never retrieved — borderline embellishment |
| 14 | How do I configure agents in CrewAI? | CrewAI not in corpus | Y | Y | — | Clean refusal |
| 15 | What did Anthropic's 2026 State of AI report conclude about agent adoption? | Document does not exist | Y | Y | — | Clean refusal |

---

## Summary

- **Answer faithfulness:** 14 / 14 questions asked = **100%** (target: 90% ✅) — every substantive answer was grounded in corpus content, and all 5 traps were refused
- **Citation accuracy:** **~1 / 9** answered questions (❌ — the headline failure)
- **Retrieval quality:** mean **1.9 / 2** (only Q6 was one-sided)
- **Trap handling:** **5 / 5** refused correctly, zero hallucinated answers
- *(Q9 pending — re-run to complete the set)*

### Failure analysis

**The dominant failure is citation integrity, not answer quality.** Chunk metadata in Pinecone contains n8n's internal UUID filenames instead of the original document names — files uploaded through the Form Trigger lose their human-readable names on the way into the Default Data Loader. The agent then does one of two things: prints the UUIDs verbatim (honest but useless — Q1, 3, 4, 5, 8), or **fabricates plausible filenames** like `langgraph-docs.md` and `langgraph.md` that don't exist (Q6, 7, 10). The second behavior is the dangerous one: a reader would trust those citations. This is a known LLM failure pattern — when the data needed to satisfy an instruction ("cite file names") is missing, the model satisfies the instruction's *form* instead of its substance.

**Fix:** re-ingest with explicit per-file metadata (split the form upload into one item per file, attach `file_name` in the Data Loader's metadata options), and tighten the system prompt to cite only the `file_name` metadata field, never to construct names. Verified fix pending re-ingestion.

**Build-time failures (fixed during development):** (1) the PDF papers crashed n8n Cloud's document loader with `DOMMatrix is not defined` (pdf.js needs browser APIs absent in the server runtime) — fixed by converting papers to markdown before ingestion; (2) Pinecone's index-creation defaults (512-dim) silently mismatch bring-your-own embeddings — the index must be created at exactly 4096 dims for Qwen3-Embedding-8B, and the same model used at ingest and query time.

### Observations

- **Refusal behavior exceeded expectations:** 5/5 traps refused with zero hallucinated content — the strict system prompt ("answer ONLY from the knowledge_base tool, reply exactly 'I couldn't find this in my corpus'") carried the day, even for highly plausible traps (Swarm, CrewAI).
- **Ambiguous questions get answered from one retrieved perspective** rather than triggering the prompt's ask-a-clarifying-question rule (Q6) — instruction-following loses to answer-momentum once relevant chunks arrive. Likely needs few-shot examples or a graded-retrieval step (Track 2).
- **The 4000-char chunks held code and explanations together** — answers about checkpointers and tool definitions came back with intact, coherent detail.
- **Metadata is part of the retrieval contract.** Embeddings and chunking worked first try after fixes; what silently broke was provenance. In Track 2 this is a one-line `metadata={"file_name": ...}` — in no-code it required redesigning the ingestion flow, a concrete illustration of the Track 1 depth-for-speed tradeoff.
