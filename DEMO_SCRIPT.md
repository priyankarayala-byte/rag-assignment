# Demo Video — Voiceover Script & Storyboard (Track 1: n8n)

Target length: ~4:30 (under the 5:00 limit). Pace: ~140 words/min — read naturally, don't rush.
Before recording: open these tabs in order — 1) PROJECT.md, 2) Finder on `corpus_upload/`, 3) n8n ingestion workflow, 4) Pinecone index page, 5) n8n chat workflow, 6) n8n chat panel.

---

## Scene 1 — Hook & one-liner (0:00–0:35)
**On screen:** PROJECT.md, one-liner visible. Optionally start on camera/title slide.

> "Hi, I'm Priyanka. I've spent over ten years as a QA automation engineer, and I'm now upskilling into agentic engineering — so for this project I built a RAG app that helps me do exactly that.
>
> Here's my one-liner: my RAG app helps a QA engineer transitioning into agentic engineering answer how-to questions about building AI agents, from a curated corpus of 45 agent-engineering documents, in an n8n chat interface, with a target of 90% faithfulness.
>
> I built this on the no-code track with n8n, with all model calls running through Nebius Token Factory."

## Scene 2 — The corpus (0:35–1:05)
**On screen:** Finder window on `corpus_upload/`, slowly scroll the 45 files.

> "The corpus is 45 documents I actually study from: Anthropic's agent-building guides and engineering blogs — including 'Building Effective Agents' — the full Model Context Protocol docs, the LangGraph documentation, and three seminal papers: ReAct, RAG, and Reflexion.
>
> Most were downloaded as native markdown straight from the publishers' docs sites. The papers started as PDFs — and that turned into my first real failure, which I'll come back to."

## Scene 3 — Ingestion workflow (1:05–2:10)
**On screen:** n8n ingestion workflow canvas. Hover each node as you mention it. Click open the splitter and embeddings nodes briefly.

> "Ingestion is one n8n workflow. A form trigger lets me drag all 45 files in manually — and re-submitting the form is my refresh strategy when these fast-moving docs change.
>
> The files flow into a Pinecone vector store node with three sub-nodes. The data loader auto-detects file types. The recursive character text splitter chunks at four thousand characters — roughly a thousand tokens — with 800 overlap. I chose that size because these docs mix prose with code examples, and smaller chunks kept separating code from its explanation.
>
> For embeddings, here's the no-code trick: this is n8n's OpenAI embeddings node, but the credential's base URL points at Nebius Token Factory, which speaks the OpenAI API. So every embedding call runs Qwen3-Embedding-8B on Nebius — that's 4096 dimensions per chunk."

## Scene 4 — Pinecone (2:10–2:30)
**On screen:** Pinecone console, `agentic-rag` index, record count visible.

> "Here's the result in Pinecone: a serverless index, 4096 dimensions, cosine similarity, holding about four hundred chunks. This is the knowledge base — n8n itself stores nothing."

## Scene 5 — Chat workflow (2:30–3:05)
**On screen:** chat workflow canvas. Open the AI Agent's system message, then the Pinecone tool node.

> "The second workflow is the Q&A side. A chat trigger feeds an AI Agent. The agent's chat model is also served by Nebius. The Pinecone index is wired in as a retrieval tool, top-k five, using the same embedding model as ingestion — query and document vectors have to live in the same space.
>
> The system prompt is strict: answer only from retrieved chunks, cite source file names after every answer, and if retrieval comes up empty, say 'I couldn't find this in my corpus' instead of guessing. I designed the refusal path before the happy path."

## Scene 6 — Live demo (3:05–4:10)
**On screen:** n8n chat panel. Type (or paste) each question, let the answer render, point at the Sources line.

> "Let's use it. First: *according to Anthropic, what's the difference between a workflow and an agent?*
> …It answers from the corpus and cites the building-effective-agents post.
>
> Something more technical: *how does an MCP server differ from an MCP client?*
> …Again, grounded, with sources.
>
> Now the most important test — a trap question: *what does my corpus say about Kubernetes autoscaling?*
> …And it refuses instead of hallucinating. For a knowledge tool, this honest 'I don't know' matters more than any correct answer."

## Scene 7 — Failures, learnings, what's next (4:10–4:40)
**On screen:** back to PROJECT.md, or your evaluation report if ready.

> "What broke along the way: the PDF papers crashed n8n's document loader with a 'DOMMatrix is not defined' error — a pdf.js incompatibility in n8n Cloud — so I converted the papers to markdown before ingestion. I also hit a dimension mismatch lesson: the Pinecone index must exactly match the embedding model at 4096.
>
> The no-code tradeoffs I'm accepting: no hybrid search, no reranking, and evals are manual. That's exactly what I'm fixing next, rebuilding this in LangChain and LangGraph with a graded retrieval step and evals as code.
>
> Fifteen-question evaluation results are in my report. Thanks for watching."

---

## Recording tips

- **Do a dry run of the 3 demo questions first** so answer latency doesn't eat your runtime; n8n keeps the chat panel responsive on the second run.
- Record with QuickTime (Cmd+Shift+5 → record selected portion) or Loom; pause recording between scenes if you need to re-stage tabs.
- If an answer is slow, narrate over the wait ("retrieval plus a 70-B model typically lands in a few seconds — within my 10-second latency budget").
- Keep the script printed/on a phone — don't read from the screen you're recording.
- Total spoken words ≈ 640 → ~4:35 at a relaxed pace. If you're over, cut Scene 4 (merge its one line into Scene 3).
