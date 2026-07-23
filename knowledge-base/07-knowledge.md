# 07 — Knowledge (Grounding & Generative Answers)

**Knowledge** gives the agent context to answer questions. It's the "R" (retrieval) in RAG. Knowledge sources are defined as `KnowledgeSourceConfiguration` and (in a cloned agent) live under `knowledge/` — including `knowledge/files/` for uploaded documents.

## Knowledge source types

| Source | `kind` (schema) | Use for |
|--------|-----------------|---------|
| **Public website** | `PublicSiteSearchSource` | Grounding on public web content (specific sites). |
| **SharePoint** | `SharePointSearchSource` | Internal docs/sites in SharePoint. |
| **Graph connectors** | `GraphConnectorSearchSource` | Enterprise data indexed via Microsoft Graph connectors. |
| **Uploaded files** | (files under `knowledge/files/`) | Documents you attach directly. |
| **Custom API / Dataverse / others** | (source-specific) | Structured/business data. |
| **Memory + org data via Microsoft IQ** | (new experience) | Persistent context + organizational knowledge. |

## How knowledge is described to the orchestrator

For `KnowledgeSourceConfiguration` files, two special leading comments matter:

```yaml
# Name: Product pricing FAQ
# Answers pricing, discount, and billing questions for premium customers
kind: PublicSiteSearchSource
# ...source config...
```

- **Line 1 `# Name:`** → display name.
- **Line 2 comment** → a **description the orchestrator uses to choose among sources**, especially when you exceed the **25-source limit** it can weigh at once. Make it specific and non-overlapping.

## Generative answers in a topic

RAG is executed by the **`SearchAndSummarizeContent`** node inside a topic. Key properties:

```yaml
- kind: SearchAndSummarizeContent
  id: search-content
  userInput: =System.Activity.Text     # what to search for
  variable: Topic.Answer               # where the answer lands
  moderationLevel: Medium              # content moderation strictness
  additionalInstructions: <extra grounding/style instructions>
  publicDataSource:
    sites:
      - "www.example.com/"
  sharePointSearchDataSource: {}       # include SharePoint source
```

- `variable: Topic.Answer` then lets you branch on `=!IsBlank(Topic.Answer)` (answer found vs not).
- **`AnswerQuestionWithAI`** is the alternative node for **general model knowledge** (no grounding source) — use carefully; grounded answers are preferred for enterprise accuracy.

## Managing knowledge files in VS Code (extension)

- **Remote Knowledge Files** window: documents you uploaded in Copilot Studio are listed but **not auto-downloaded** — click a name to download when you need it (you get a success notification).
- **To upload new files**: drop them into the `knowledge/files/` folder, then **Apply** — they're uploaded via the agent contents upload feature.

## The "Use general knowledge" setting

Agents have a setting controlling whether the built-in knowledge tool may use the model's **general knowledge** (vs only your sources). Note for multi-agent: child/connected agents **respect the main agent's** "Use general knowledge" setting; they won't use general knowledge as a source, though the underlying model may still use latent knowledge when phrasing questions/messages.

## Design guidance

- Prefer **grounded** answers (specific sources) over general knowledge for accuracy/compliance.
- Keep source **descriptions distinct** so orchestration routes correctly (mind the 25-source weighting).
- Set an appropriate **moderation level**.
- Combine: generative answers for open Q&A + **topics** for deterministic flows + **tools** for actions.
- For rapidly changing content, RAG over a live source beats baking answers into topics.

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
