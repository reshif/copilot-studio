# 11 — Code-First Framework & Our Reusable Build Workflow

This file turns everything above into a **repeatable way to build agents** in VS Code — both by leveraging Microsoft's open `skills-for-copilot-studio` framework and by defining our own conventions.

## The `microsoft/skills-for-copilot-studio` framework

An open, **code-first** framework for building Copilot Studio agents from **YAML**, plus skills for validation, testing, and troubleshooting. Its central principle: **mandatory skill delegation** — author/troubleshoot agents don't hand-write YAML or run scripts blindly; they invoke schema-aware skills, which **prevents hallucinated `kind`s, missing required fields, and validation errors**. This is exactly the discipline we want when using an AI assistant (Claude Code / GitHub Copilot) to author agents.

Skills are invoked as `/copilot-studio:<skill-name>`.

### Author skills (create YAML)
- `add-knowledge` — configure knowledge sources (public sites, SharePoint, Graph connectors, custom APIs).
- `add-global-variable` — persistent conversation-level variables with AI-visibility controls.
- `add-action` / `edit-action` — integrate/modify connector actions (M365, Teams, Outlook, SharePoint); change input/output + connection modes.
- `add-adaptive-card` — form/info/confirmation cards with input validation.
- `new-topic` — create topics with a chosen trigger type.
- `add-node` — add nodes (`SendActivity`, `Question`, `BeginDialog`, `ConditionGroup`, `SetVariable`, `HttpRequest`).
- `add-generative-answers` — RAG (`SearchAndSummarizeContent`) or general knowledge (`AnswerQuestionWithAI`).
- `add-other-agents` — child/connected multi-agent patterns.
- `edit-agent` — instructions, display names, config flags.
- `edit-triggers` — trigger phrases and `modelDescription`.

### Test skills
- `chat-with-agent` — utterances via Copilot Studio Client SDK (published agents only).
- `run-tests` — batch (Kit mode via Dataverse API) or offline eval (Eval mode from CSV).
- `directline-chat` — DirectLine v3 REST API, OAuth supported.

### Troubleshoot skills
- `known-issues` — search a GitHub KB of documented problems/mitigations.
- `validate` — check YAML vs schema (unknown kinds, duplicates, missing fields, Power Fx errors).
- `lookup-schema` — query kind definitions & property rules.
- `list-kinds` — valid `kind` values by context (dialog/trigger/action).
- `list-topics` — enumerate topics by scanning `topics/*.mcs.yml`.

### Validation/infra tooling
- `schema-lookup.bundle.js` — validates against `bot.schema.yaml-authoring.json`. Commands: `validate <file>`, `lookup <name>`, `kinds`, `resolve <name>`.
- `connector-lookup.bundle.js` — discover available connectors/operations.
- `chat-with-agent.bundle.js`, `run-tests.js`, `directline-chat.bundle.js` — testing implementations.

### Framework workflow stages
1. **Author** — skills create schema-valid YAML.
2. **Push** — deploy to an environment (child agents need a pre-push before adding knowledge).
3. **Test** — `chat-with-agent` / batch testing.
4. **Troubleshoot** — `validate` + KB lookups.
5. **Publish** — expose via published endpoints for testing.

> Adopt this even if we don't use the exact repo: **always validate against the schema before Apply**, and let the AI assistant call schema-aware helpers rather than free-writing YAML.

## Our recommended build workflow (VS Code + AI assistant)

1. **Bootstrap**: create a minimal agent in the Copilot Studio portal → **Clone** to a Git-tracked folder → `git init`.
2. **Understand the skeleton**: read `agent.mcs.yaml` (instructions, `grounding.GenerativeActionsEnabled`), `settings.mcs.yml`, and folder layout ([04](04-agent-definition-yaml-schema.md)).
3. **Author iteratively**, one component per commit:
   - Instructions → knowledge → tools → topics → triggers → (child/connected agents).
   - Use AI assistant to draft, but **validate every file** against the schema; keep IDs unique; prefix Power Fx with `=`.
4. **Apply → test** in the Copilot Studio test pane; inspect the **activity map** to verify routing.
5. **Evaluate** with test sets; fix regressions.
6. **Commit + PR**; promote via **solutions/pipelines** ([10](10-alm-publishing-testing.md)); **Publish** in prod.

## Reusable framework conventions (our house style)

**Repo/folder**
- One Git repo per agent (or per solution of related connected agents).
- Keep the cloned structure; add `/knowledge-base` (this KB), `/docs`, and `/tests`.

**Naming** ([04](04-agent-definition-yaml-schema.md))
- Files: kebab-case + type suffix (`create-ticket.tool.yaml`, `escalate.topic.yaml`).
- IDs/variables: camelCase, descriptive, no abbreviations.

**Descriptions are product** (they drive orchestration)
- Every tool/topic/knowledge source/agent gets a **distinct, specific description** / `modelDescription`. Keep total routable choices **< ~30–40** per agent.

**Determinism where it matters**
- Compliance/escalation/payment/data-collection → **authored topics**.
- Open Q&A → **generative answers (RAG)**.
- System-of-record actions → **tools/flows**.

**Quality gates before Apply**
- No Problems-pane errors; Power Fx `=`-prefixed; unique node IDs; no `TODO`/`_REPLACE`; schema-valid kinds.
- Preview/Get latest remote changes first (Apply is blocked otherwise).

**Safety**
- Never Apply straight to prod; dev → test → prod via solutions.
- Remember Apply ≠ Publish. Connection references/env variables are per-environment.

## Reusable agent blueprint (starter mental template)

```text
agent.mcs.yaml            # identity + instructions; GenerativeActionsEnabled: true
settings.mcs.yml          # model, general-knowledge toggle, moderation
knowledge/                # 1–N well-described sources (< 25 weighted)
  files/
topics/
  greeting.topic.yaml     # OnConversationStart welcome
  fallback.topic.yaml     # OnUnknownIntent -> SearchAndSummarizeContent (RAG)
  escalate.topic.yaml     # deterministic human handoff
actions/                  # tools: flows / REST / connectors (distinct descriptions)
trigger/                  # autonomous triggers (WorkflowExternalTrigger) if needed
agents/                   # child agents when grouping subtasks
adaptive-cards/           # rich input/confirmation cards
```

## Open items to verify when we start building

- Exact current set of extension commands/settings (check the installed version + GitHub repo).
- Whether the extension yet supports **create-from-scratch** (was "coming" at GA).
- New-experience YAML parity vs classic (the extension currently centers on classic-style definitions).
- Precise schema of newer `kind`s as the schema file evolves — always `validate`/`lookup-schema` against the shipped `bot.schema.yaml-authoring.json`.

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
