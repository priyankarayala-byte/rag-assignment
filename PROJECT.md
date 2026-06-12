# Agentic Engineering Study Assistant — RAG Project (Week 2)

## One-liner

My RAG app helps **a software engineer who is transitioning into agentic engineering** answer **conceptual and how-to questions about building AI agents** from **a curated corpus of 45 agent-engineering documents (Anthropic agent guides + engineering blogs, MCP documentation, LangGraph docs, 3 seminal papers)** in **an n8n chat interface** with **90% faithfulness and <10s latency**.

## Framework

| Field | Decision |
|---|---|
| **Use case** | "How do I build X / what is Y in agentic engineering?" — asked by me while studying, surfaced in n8n's chat UI. |
| **Corpus** | 45 docs in 4 groups: Anthropic docs + engineering blog posts (18), MCP documentation (13), LangGraph docs (11), seminal papers as PDFs (3: ReAct, RAG, Reflexion). Markdown + PDF, English. Source of truth: the publishers; I curate snapshots. |
| **Ingestion + cleaning** | Docs fetched as native markdown where the publisher serves it (docs.langchain.com, modelcontextprotocol.io, platform.claude.com all serve `.md`); blog posts converted HTML→markdown via Jina Reader; papers kept as PDF. Stored in a GitHub repo; n8n ingests from there. |
| **Freshness** | Manual re-run of the ingestion workflow monthly — agent tooling docs change fast; snapshot date recorded in this repo's git history. |
| **Chunking + embedding** | Recursive character splitter, ~1000 tokens (≈4000 chars) with ~200-token overlap — these docs mix prose with code blocks, and smaller chunks split code from its explanation. Embeddings: `Qwen/Qwen3-Embedding-8B` via **Nebius Token Factory** (4096-dim) — satisfies the mandatory Nebius requirement. |
| **Retrieve** | Pinecone (serverless free tier), dense retrieval, cosine, top-k 5. Known tradeoff of the n8n track: no true hybrid (BM25+dense) or reranking — documented here, to be addressed in the Track 2 (LangChain/LangGraph) rebuild. |
| **Refusal path** | Agent system prompt: answer ONLY from retrieved chunks, cite the source document for every claim, and reply "I couldn't find this in my corpus" when retrieval comes back empty/irrelevant. Tested with trap questions in the eval. |

## Build track

- **Now:** Track 1 — n8n Cloud (no-code)
- **Later:** Track 2 — LangChain + LangGraph rebuild of the same scope

## Deliverables checklist

- [ ] Working Q&A bot in n8n (ingestion workflow + chat workflow)
- [ ] 15-question evaluation report (incl. ambiguous, multi-doc, and unanswerable questions) with failure analysis
- [ ] Google Doc: overview, corpus, prompts, iterations, learnings
- [ ] Demo video (≤5 min)
- [ ] GitHub repo: corpus + exported n8n workflow JSONs + this doc
- [ ] Submit: https://forms.gle/3vj27gwoxw2xk9B7A
