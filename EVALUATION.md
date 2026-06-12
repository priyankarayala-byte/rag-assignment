# Evaluation Report — Agentic Engineering Study Assistant (Track 1: n8n)

**Method:** 15 questions in 3 buckets (5 straightforward, 5 ambiguous/multi-document, 5 unanswerable traps), asked one at a time in the n8n chat panel. For each question I record what was retrieved (the cited file names), whether the answer was faithful to the corpus, and the failure mode if any.

**Scoring rubric**
- **Retrieval (0–2):** 2 = cited sources contain the answer · 1 = partially relevant sources · 0 = wrong/no sources
- **Faithful (Y/N):** every claim in the answer is supported by the corpus (for traps: a refusal counts as faithful; any invented answer = N)
- **Citation (Y/N):** answer lists the correct source file name(s)
- **Failure mode (if failed):** `retrieval-miss` (right doc never retrieved) · `generation-drift` (right doc retrieved, answer strayed) · `over-refusal` (answer existed but bot refused) · `hallucination` (trap answered confidently)

---

## Bucket A — Straightforward (single-document answers)

| # | Question | Expected source(s) | Retrieval (0–2) | Faithful | Citation | Failure mode | Notes |
|---|---|---|---|---|---|---|---|
| 1 | According to Anthropic, what is the difference between a workflow and an agent? | blog-building-effective-agents.md | | | | | |
| 2 | What are the three core building blocks an MCP server can expose to clients? | mcp-server-concepts.md | | | | | |
| 3 | What is the ReAct pattern, and what two capabilities does it interleave? | paper-react-reasoning-acting-llms.md | | | | | |
| 4 | What is a checkpointer in LangGraph and what does it enable? | langgraph-persistence.md | | | | | |
| 5 | What does Anthropic recommend for writing good tool descriptions for agents? | blog-writing-tools-for-agents.md, anthropic-define-tools.md | | | | | |

## Bucket B — Ambiguous or multi-document

| # | Question | Expected source(s) | Retrieval (0–2) | Faithful | Citation | Failure mode | Notes |
|---|---|---|---|---|---|---|---|
| 6 | How do agents remember things? *(ambiguous: Anthropic's memory tool vs LangGraph memory — ideally synthesizes or asks which)* | anthropic-memory-tool.md, langgraph-add-memory.md | | | | | |
| 7 | What's the best way to manage context in a long-running agent? | blog-effective-context-engineering-for-ai-agents.md, anthropic-context-windows.md, anthropic-context-editing.md | | | | | |
| 8 | How should I debug my MCP integration? | mcp-debugging.md, mcp-inspector.md | | | | | |
| 9 | How does Reflexion build on ReAct? *(spans two papers)* | paper-reflexion-verbal-reinforcement.md, paper-react-reasoning-acting-llms.md | | | | | |
| 10 | When should I use multiple agents instead of a single agent? | blog-multi-agent-research-system.md, blog-building-effective-agents.md | | | | | |

## Bucket C — Unanswerable traps (correct behavior = refusal)

| # | Question | Why it's a trap | Refused? | Faithful | Failure mode | Notes |
|---|---|---|---|---|---|---|
| 11 | What does the corpus say about Kubernetes autoscaling? | Out of domain entirely | | | | |
| 12 | What does OpenAI's Swarm framework recommend for agent handoffs? | Plausible topic, but Swarm is not in the corpus | | | | |
| 13 | What is the per-token price of Claude Opus? | Pricing appears nowhere in the corpus | | | | |
| 14 | How do I configure agents in CrewAI? | CrewAI is not in the corpus | | | | |
| 15 | What did Anthropic's 2026 State of AI report conclude about agent adoption? | Document does not exist | | | | |

---

## Summary (fill after running all 15)

- **Faithfulness:** ___ / 15 = ___ % (target: 90%)
- **Retrieval quality:** mean ___ / 2
- **Citation accuracy:** ___ / 10 (Buckets A+B)
- **Trap handling:** ___ / 5 refused correctly
- **Median latency (rough):** ___ s (target: <10 s)

### Failure analysis

*(For every failed row, 2–3 sentences: what was retrieved, why it failed — retrieval-miss vs generation-drift vs hallucination — and what would fix it: e.g. hybrid search for exact-term queries, reranking, smaller chunks, stricter prompt. Failures observed during the build also belong in the story: the PDF loader crash (`DOMMatrix is not defined` on n8n Cloud — fixed by converting papers to markdown pre-ingestion) and the embedding-dimension/index mismatch lesson (index must be exactly 4096 for Qwen3-Embedding-8B).)*

### Observations

*(2–3 honest observations, e.g.: dense-only retrieval struggles with exact acronyms; ambiguous questions get answered from one perspective without asking for clarification; chunk size 4000 kept code examples intact.)*
