# Reference — File & Attachment Handling

**The core gotcha:** a Copilot Studio **custom agent does not natively process user-uploaded files** the way Microsoft 365 Copilot does. Uploaded files are essentially **invisible** to the agent unless you build handling. By default (per official docs), if a user sends an attachment, the agent replies it **can only process text** — this applies across channels, even ones whose UI supports attachments (Teams, Direct Line).

> This section blends the **official constraint + system variable** (Microsoft Learn) with a **practitioner-sourced solution ladder** (George Karapetyan, Medium — secondary). Verify current behavior before committing to an approach.

## What IS available natively
- **`System.Activity.Attachments`** (type **table**) — the system variable holding files the user provided ([variables-and-entities.md](variables-and-entities.md)). You can detect/loop over it in a topic.
- **Knowledge files** you (the maker) upload are indexed for RAG — that's *maker* content, not *end-user* uploads (see [knowledge.md](knowledge.md)).
- **Adaptive Card** inputs, and **prompt tools** that accept images ([prompts-and-agent-flows.md](prompts-and-agent-flows.md)).

## The solution ladder (no-code → full custom)
Cost/complexity rise as you go down; control and robustness rise too.

| # | Approach | Skill | How it works | Limits / notes |
|---|----------|-------|--------------|----------------|
| 1 | **Custom topic + built-in actions** | No-code | Trigger when `System.Activity.Attachments` has files → loop, check type (PDF/PNG/JPEG) → send to **prompt** action nodes; use **OneDrive conversion** actions for unsupported formats. | ~50-page cap, few file types, **no search**, high latency. Good for MVP / acknowledgment. |
| 2 | **Azure AI Foundry agent** | Low-code | Foundry agent auto-builds a **vector store**; files index automatically and **persist ~7 days/session**; call it over **HTTPS** from a topic; capture **thread/file IDs** for follow-ups. | Extra Azure cost + setup (keys/endpoints/permissions). Robust doc handling. |
| 3 | **Azure AI Search index + indexer** | Low/pro-code | Blob Storage as source; custom **schema enforcing user-level separation**; upload → trigger indexer → query (filtered by **user ID**) → feed generative nodes. | Coding + maintenance; schema design. Best for **production, long-term storage, privacy**. |
| 4 | **PromptFlow (Azure AI Foundry)** | Low-code (+opt. Python) | Drag-drop flow loads files into a vector store (+ optional Python blocks) → deploy as **HTTPS endpoint** → call from Copilot Studio. | Compute cost; steeper for non-devs. Flexible. |
| 5 | **Azure Logic Apps** | Pro-code | HTTP-triggered Logic App receives the upload → built-in connectors / **Azure Functions** process → return to the agent. | Higher setup/maintenance; dev expertise. Good for cross-ecosystem automation. |
| 6 | **Custom API + vector store** | Full custom | Your backend (Functions/servers) + LangChain/Pinecone/Qdrant/Milvus; own indexing/search; integrate via HTTP. | Full ownership. For strong dev teams / unique needs. |

## Decision guidance
- **Just acknowledge / light parsing of a PDF** → Approach 1 (topic + prompt/OneDrive).
- **Real document Q&A, minimal infra** → Approach 2 (Foundry agent) — note the ~7-day persistence.
- **Production, per-user privacy, durable storage** → Approach 3 (AI Search with user-ID filtering) or 6.
- **Custom pipelines / Python processing** → Approach 4 or 5.
- Always pass results back into a **generative-answers node** or the tool's completion so the agent can summarize with the user's context.

## Cross-references
- Attachment channel limitation & "text only" reply → [publishing-security-limits.md](publishing-security-limits.md).
- `System.Activity.Attachments` and voice input types → [variables-and-entities.md](variables-and-entities.md).
- Calling external endpoints → HTTP node ([node-types.md](node-types.md)) or REST/flow tools ([tools.md](tools.md), [prompts-and-agent-flows.md](prompts-and-agent-flows.md)).
