# Reference — Knowledge (complete)

Knowledge grounds **generative answers**. Add at **agent level** (Knowledge page) or **topic level** (generative-answers node). All agent-level sources feed the generative-answers node in the **Conversational boosting** system topic (classic).

## Supported knowledge sources

| Source | Int/Ext | How it works | Generative-mode inputs | Classic-mode inputs | Auth |
|--------|---------|--------------|------------------------|---------------------|------|
| **Public website** | External | Bing search restricted to provided sites | 25 websites | 4 public URLs | None |
| **Documents** (uploaded to Dataverse) | Internal | Searches doc contents | All documents | Limited by Dataverse file storage | None |
| **SharePoint** | Internal | GraphSearch over a SharePoint URL | 25 URLs | 4 URLs per node | Agent user's Entra ID |
| **Dataverse** | Internal | RAG technique in Dataverse | Unlimited | 2 sources (≤15 tables each) | Agent user's Entra ID |
| **Enterprise data via connectors** | Internal | Org data indexed by Microsoft Search | Unlimited | 2 per agent | Agent user's Entra ID |

Plus **unstructured data** sources: OneDrive files/folders, uploaded SharePoint files/folders, Salesforce, Confluence, ServiceNow, Zendesk (all require **user-level auth**; ALM not supported for these — imported agents don't auto-process them; 4–6h sync).

**User auth for knowledge** = the agent surfaces only content the asking user can access.

Not supported *inside generative-answers nodes* directly: Bing Custom Search, Azure OpenAI, Custom Data — use the node's **Classic data** option for those.

## Search modes

### Classic orchestration
- Uses the **Conversational boosting** system topic. Combined source caps: Azure OpenAI **5**, Bing Custom Search config IDs **2**, custom data sources **3**, Dataverse **2** (≤15 tables each), SharePoint URLs **4**, uploaded files **unlimited**, website URLs **4**.
- Can also embed a **generative-answers node** for specific intents (same caps).
- Supports custom data sources.

### Generative orchestration
- Agent proactively searches knowledge. When there are **>25 distinct sources**, an internal GPT model **filters** by description (so descriptions matter). **Uploaded files don't count** toward the 25.
- **Does NOT** support custom data or Bing Custom Search as sources (embed them in a generative-answers node instead).
- **Ignores** the Conversational boosting system topic customizations.

## Agent Generative-AI knowledge settings

- **Allow ungrounded responses** — on ⇒ agent may answer from the model's general knowledge with no source/tool used. Off ⇒ blocks any turn where no source/tool was used → fallback topic fires. (Off requires an in-text **citation** to return a knowledge answer; models sometimes omit citations → intermittent "no info found". Mitigate by instructing the model to always cite, avoiding format constraints like "JSON only", and adding `ContentLocation`+`Title` to custom data.)
- **Use information from the web / Web Search** — on ⇒ searches all Bing-indexed public sites (via Grounding with Bing Search), interleaved with your public-website sources. Requires generative orchestration.
- **Moderation** — **Lowest → Highest** (default **High**). Higher = fewer but safer answers. Set at agent level (Generative AI settings), topic level (generative-answers node), or prompt level; **topic-level takes precedence** at runtime.
- **Tenant graph grounding with semantic search** — uses M365 semantic index for better retrieval; requires **Authenticate with Microsoft** and a M365 Copilot license in the tenant; extra cost; enables SharePoint/connector files up to **200 MB** (else 512 MB for PDF/PPTX/DOCX). Slight latency increase possible.
- **Official sources** — mark a trusted source so the agent uses it without verification (response starts with a distinctive indicator). **Incompatible with generative orchestration** (turn generative off to use it).

## Citations
- Returned inline for grounded answers; **can't** be used as inputs to other tools/actions.
- **Teams**: max **20** citations per response; title ~80 chars, snippet ~480 chars.
- **Customized** answers (you render via variable/Adaptive Card) don't auto-add citations — render them yourself.
- Hyperlinks inside source docs render as **plain text**.

## Generative-answers node (topic-level) — YAML
See `SearchAndSummarizeContent` in [node-types.md](node-types.md). Key knobs: `userInput`, `variable`, `moderationLevel`, `additionalInstructions`, `publicDataSource.sites`, `sharePointSearchDataSource`. Use for specific intents (not just fallback), and to reach Classic-data sources under generative orchestration.

## Key limits (see [publishing-security-limits.md](publishing-security-limits.md) for the full list)
- **500** knowledge sources per agent (all types); **500** files; **512 MB**/file (images only in PDFs).
- SharePoint (integrated): **25 site URLs** in generative mode; modern pages only; DOC/DOCX/PPT/PPTX/PDF; <7 MB files without a M365 Copilot license in-tenant; lists ≤15 (≤35k rows/list, ≤120k total, first 2,048 rows on list queries).
- Dataverse: **2 sources**/agent, **15 tables**/source; standard/activity tables; READ permission required; synonyms/glossaries supported.
- OneDrive / uploaded-SharePoint: 1,000 files, 50 folders, 10 subfolder levels, 512 MB/file; no *confidential/highly-confidential* sensitivity-labeled or password-protected docs; no glossaries/synonyms; **ALM not supported**.
- Page-level citations: PDF only (Upload files → SharePoint path); others fall back to document-level.

## Build guidance
- Prefer **grounded** answers; keep **Allow ungrounded** off for enterprise accuracy unless you specifically need general knowledge.
- Write **distinct, specific** source descriptions (matters most past 25 sources).
- Instruct the model to **always cite** to avoid withheld answers.
- Use **semantic search** when you have M365 Copilot licensing for better SharePoint retrieval.
- For real-time public info, enable **Web Search**; for authoritative internal docs, consider **Official sources** (classic only).
