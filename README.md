# Agentic Engineering Study Assistant — RAG (Week 2 Project)

A RAG-powered Q&A assistant over a curated corpus of agent-engineering documentation (Anthropic agent guides, MCP docs, LangGraph docs, and the ReAct/RAG/Reflexion papers). Built on the **no-code track with n8n**, embeddings and generation served by **Nebius Token Factory**, vectors in **Pinecone**.

> My RAG app helps a QA automation engineer transitioning into agentic engineering answer conceptual and how-to questions about building AI agents, from a curated corpus of 45 agent-engineering documents, in an n8n chat interface, with 90% faithfulness and <10s latency.

## Repo contents

| Path | What it is |
|---|---|
| [PROJECT.md](PROJECT.md) | Scope: one-liner, full RAG framework table, deliverables checklist |
| [SUBMISSION_DOC.md](SUBMISSION_DOC.md) | Project documentation: architecture, prompts, iterations, failures, learnings |
| [EVALUATION.md](EVALUATION.md) | 15-question evaluation report with retrieval scores and failure analysis |
| [DEMO_SCRIPT.md](DEMO_SCRIPT.md) | Storyboard + voiceover script for the demo video |
| [workflows/](workflows/) | Exported n8n workflow JSONs (ingestion + chat) |
| [corpus/](corpus/) | Source corpus, organized by origin (see its README for sources/licensing) |
| [corpus_upload/](corpus_upload/) | Flattened, ingestion-ready corpus (45 markdown files; PDFs pre-converted) |

## Architecture

```
Ingestion:  Form Trigger ─→ Pinecone Vector Store (Insert)
                               ├─ Default Data Loader (binary, auto-detect)
                               │    └─ Recursive Character Text Splitter (4000 / 800)
                               └─ Embeddings: Qwen/Qwen3-Embedding-8B @ Nebius (4096-dim)

Q&A:        Chat Trigger ─→ AI Agent (Nebius chat model, Simple Memory)
                               └─ Tool: Pinecone retrieval (top-k 5, same embeddings)
```

The Nebius integration uses n8n's OpenAI nodes with the credential Base URL set to `https://api.tokenfactory.nebius.com/v1` (Nebius is OpenAI-API-compatible).

## Reproduce

1. Create a Pinecone serverless index: dimension **4096**, metric **cosine**.
2. Import the two JSONs from `workflows/` into n8n; attach your own Nebius + Pinecone credentials.
3. Execute the ingestion workflow and upload the files from `corpus_upload/` via the generated form.
4. Open the chat on the Q&A workflow and ask away.

Corpus content belongs to its original publishers (Anthropic, the MCP project, LangChain, arXiv authors); snapshots included for educational use in this assignment.
