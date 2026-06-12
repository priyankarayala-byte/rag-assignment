# n8n Workflow Exports

Export from n8n: open each workflow → ⋯ menu → Download.

- `ingestion-workflow.json` — Form Trigger → Pinecone Insert (Nebius embeddings, recursive splitter 4000/800)
- `chat-workflow.json` — Chat Trigger → AI Agent → Pinecone retrieval tool (top-k 5)

Exports contain no credentials/API keys.
