fin# Reference — Custom Knowledge Sources (`OnKnowledgeRequested`)

Bring **your own search endpoint** (custom API, enterprise search, Azure AI Search) into generative answers while keeping full control of the query. Any topic using the **`OnKnowledgeRequested`** trigger acts as a custom knowledge source and contributes results.

> ⚠️ **Code-only.** This trigger can be configured **only in YAML/code view** — there is no visual designer support. This makes it a natural fit for our VS Code code-first workflow ([../03-vscode-extension.md](../03-vscode-extension.md)).

## When it fires
- When the **orchestrator** decides knowledge retrieval is needed, **and**
- When a **generative answers node** is explicitly invoked.

## Special system variables (only available in these topics)
| Variable | Purpose |
|----------|---------|
| `System.SearchQuery` | Context-aware, **rewritten** query optimized for **semantic** search |
| `System.KeywordSearchQuery` | Rewritten query optimized for **keyword** search engines |
| `System.SearchResults` | Where **you** store the formatted knowledge snippets |

Copilot Studio rewrites queries using conversation history, preserving multi-turn context — **always use the rewritten variable**, not the raw user text.

### Query rewriting in action
- Turn 1: *"What's our official data retention period for customer records?"*
- Turn 2: *"Does it change for financial information?"*
- Turn 3: *"And are there exceptions?"*

**Rewritten:** *"exceptions to data retention policy customer and financial data retention exceptions regulatory exemptions policy exception handling compliance guidelines"* — it resolves "there", pulls forward customer + financial context, and adds enterprise policy vocabulary.

## Step 1 — Create the trigger
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnKnowledgeRequested
  id: main
  intent: {}
  actions:
    # actions go here
inputType: {}
outputType: {}
```

## Step 2 — Call your search endpoint
```yaml
- kind: HttpRequestAction
  id: searchRequest
  url: ="https://search-api.contoso.com/search?q=" & System.KeywordSearchQuery
  response: Topic.searchResults
  responseSchema:
    kind: Record
    properties:
      query: String
      results:
        type:
          kind: Table
          properties:
            snippet: String
            title: String
            url: String
```
> You can use anything that reaches a search endpoint — raw HTTP, **custom connectors**, built-in connectors (e.g. **Azure AI Search**), or **agent flows**. Tip: build the response schema in the UI via **From sample data → Get schema from sample JSON**, which generates the YAML for you.

## Step 3 — Transform results into the expected shape
Copilot Studio expects each result to have:
- **`Content`** — the snippet/excerpt (required)
- **`ContentLocation`** — URL (optional, but needed for citations)
- **`Title`** — result title (optional)

```yaml
- kind: SetVariable
  id: setSearchResults
  variable: System.SearchResults
  value: |-
    =ForAll(Topic.searchResults.results,
    {
      Content: snippet,
      ContentLocation: url,
      Title: title
    })
```
`ForAll` maps each API result into the expected record shape, and the table is assigned to **`System.SearchResults`**, which the platform uses to generate the grounded answer.

## Considerations & limits
- **15-snippet cap** — Copilot Studio uses up to **15 snippets** from `System.SearchResults`. If your API returns more: implement **relevance scoring**, limit the API response to 15, or **sort by relevance** before transforming.
- **Multiple `OnKnowledgeRequested` topics are allowed** — each can query a different backend, and Copilot Studio **invokes them all simultaneously**. ⚠️ The **15-result cap applies across all of them combined** (Topic A returns 10 + Topic B returns 8 ⇒ only the top 15 are used).
- **Recommendations:** sort/score before returning · keep response schemas consistent · use clear topic names and descriptions (helps relevance filtering with large result sets).

## Why this matters
It's the supported way to:
- Ground answers in a **proprietary index** the platform doesn't natively support.
- **Filter or re-rank** results before the model sees them.
- **Merge external data** into the knowledge response.
- Keep **security trimming/logic** in your own service.

Include `ContentLocation` and `Title` so the model can **cite** — critical when *Allow ungrounded responses* is off, or grounded answers get withheld ([knowledge.md](knowledge.md)).

## Related
[knowledge.md](knowledge.md) · [generative-orchestration.md](generative-orchestration.md#control-nodes--triggers-for-generative-agents) · [node-types.md](node-types.md) · [../guidance/agent-design-playbook.md](../guidance/agent-design-playbook.md)
