# Week 2 Project: Agentic Engineering Study Assistant (RAG, Track 1 — n8n)

**Author:** Priyanka Rayala · **Date:** June 2026
**Demo video:** *(link)* · **GitHub:** *(link)* · **Evaluation report:** see EVALUATION.md / *(link)*

## 1. Project overview

I spent 10+ years as a QA automation engineer and am upskilling into agentic engineering. Instead of a generic demo corpus, I built a RAG assistant over the material I'm actually studying — so the project compounds with my learning.

**One-liner:** My RAG app helps a QA automation engineer transitioning into agentic engineering answer conceptual and how-to questions about building AI agents, from a curated corpus of 45 agent-engineering documents, in an n8n chat interface, with 90% faithfulness and <10s latency.

**Framework decisions** (full table in PROJECT.md):

| Layer | Decision | Why |
|---|---|---|
| Corpus | 45 docs: Anthropic agent guides + engineering blogs (18), MCP docs (13), LangGraph docs (11), papers — ReAct, RAG, Reflexion (3) | The exact material I study; mixed doc types exercise ingestion |
| Ingestion | Native markdown from publishers' docs sites; blogs HTML→md via Jina Reader; papers PDF→md locally; manual upload via n8n Form Trigger | n8n Cloud can't read local disks; the form makes re-ingestion a one-click refresh |
| Chunking | Recursive character splitter, 4000 chars (~1000 tokens), 800 overlap | Docs mix prose and code; smaller chunks split code from its explanation |
| Embeddings | Qwen/Qwen3-Embedding-8B via **Nebius Token Factory** (4096-dim) | Satisfies the mandatory Nebius requirement at the embedding layer |
| Store / retrieval | Pinecone serverless, cosine, dense top-k 5 | Free tier; dense-only is a documented n8n-track tradeoff |
| Generation | Nebius-hosted chat model via n8n's AI Agent node | Single Nebius credential serves both embedding and generation |
| Refusal path | System prompt: answer only from retrieved chunks, cite file names, otherwise reply "I couldn't find this in my corpus" | Designed the refusal before the happy path |

## 2. Architecture

Two n8n workflows:

1. **Ingestion:** Form Trigger (multi-file upload) → Pinecone Vector Store (Insert) with sub-nodes: Default Data Loader (binary, auto-detect) → Recursive Character Text Splitter (4000/800) → Embeddings OpenAI node pointed at Nebius. Result: ~400 chunks in a 4096-dim index.
2. **Q&A:** Chat Trigger → AI Agent (Nebius chat model + Simple Memory) → Pinecone as a retrieval tool (`knowledge_base`, top-k 5, same embedding model as ingestion).

**The key no-code trick:** n8n has no Nebius node. Because Nebius Token Factory exposes an OpenAI-compatible API, I used n8n's OpenAI nodes with the credential's Base URL set to `https://api.tokenfactory.nebius.com/v1` — every "OpenAI" call in the workflows actually runs on Nebius.

## 3. Prompts used

**Agent system prompt (final version):**

> You are a study assistant for an engineer learning agentic AI engineering. Answer questions using ONLY the information returned by the knowledge_base tool. Always call knowledge_base before answering.
> 1. Ground every claim in the retrieved documents. Never answer from your own general knowledge.
> 2. After each answer, list the source file names you used.
> 3. If the retrieved content does not contain the answer, reply exactly: "I couldn't find this in my corpus." Then suggest what document type might cover it. Do not guess.
> 4. If the question is ambiguous, ask one clarifying question instead of answering.
> 5. Keep answers concise; quote key definitions verbatim when helpful.

**Retrieval tool description** (matters as much as the system prompt — it's how the agent decides to search):

> Searches a curated corpus of agentic engineering documentation: Anthropic agent guides, MCP docs, LangGraph docs, and research papers (ReAct, RAG, Reflexion). Always use this before answering.

I also used AI assistance (Claude Code) throughout the build for corpus curation, n8n configuration guidance, and debugging — examples of prompts I gave it: scoping the use case with the assignment framework, "why can't I manually upload the parent folder on n8n ui?", and pasting raw error messages like `DOMMatrix is not defined` for diagnosis.

## 4. Iterations and failures (and fixes)

1. **PDF ingestion crash.** First full ingestion run failed with `DOMMatrix is not defined`. Root cause: n8n Cloud's Default Data Loader uses pdf.js, which requires a browser API absent from the server runtime — the 3 arXiv PDFs killed the run while all 42 markdown files were fine. **Fix:** converted the papers to markdown locally (pypdf) and re-ingested 45 pure-text files. **Lesson:** in no-code tools you can't patch the runtime; you adapt the data instead.
2. **Embedding-dimension discipline.** Pinecone's index creation defaults (512-dim, or "integrated embedding model" presets) silently conflict with bring-your-own embeddings. The index had to be created with custom settings at exactly 4096 to match Qwen3-Embedding-8B, and the identical model must be used at ingestion *and* query time.
3. **Credential redirection.** n8n auto-attached its "free OpenAI credits" credential to the embeddings node. Using it would have both violated the Nebius requirement and produced 1536-dim vectors that the 4096-dim index would reject — caught before ingestion.
4. **Folder-upload assumption.** I expected to upload a folder of documents; n8n has no document library — files only exist as binary data inside a workflow execution, and persistence is the vector store's job. The Form Trigger (multiple files) became both the ingestion UI and the freshness mechanism: refresh = resubmit the form (with Clear Namespace on, making re-ingestion idempotent).

## 5. Evaluation summary

15 questions (5 straightforward, 5 ambiguous/multi-doc, 5 unanswerable traps). Full per-question scores and failure analysis: EVALUATION.md.

- Faithfulness: ___% (target 90%)
- Retrieval quality: ___/2 mean · Citation accuracy: ___/10 · Traps refused: ___/5
- Headline finding: *(fill in — e.g. "dense-only retrieval missed exact-acronym queries; reranking/hybrid is the first Track 2 upgrade")*

## 6. Learnings and observations

- **Chunking and the embedding model are one decision, not two** — chunk size, vector dimension, and index config all have to agree, and the failure when they don't is at insert time, not build time.
- **The refusal path is a prompt + an eval, not a feature toggle.** n8n gives no confidence threshold; refusal lives entirely in the system prompt, so trap questions in the eval are the only way to know it works.
- **No-code's ceiling is real but further than expected.** The OpenAI-compatibility trick got a non-supported provider (Nebius) working everywhere; what I genuinely couldn't do in n8n: hybrid retrieval, reranking, evals as code.
- **QA instincts transfer directly to RAG evaluation** — designing trap questions, failure taxonomies (retrieval-miss vs generation-drift vs hallucination), and idempotent re-runs are test-engineering habits applied to an AI system.

## 7. Next steps (Track 2)

Rebuild the same scope in LangChain + LangGraph against the same Pinecone index: explicit retrieve → grade-documents → generate/refuse graph, hybrid retrieval (BM25 + dense), reranking, and the same 15-question eval run as code for a direct Track 1 vs Track 2 comparison.
