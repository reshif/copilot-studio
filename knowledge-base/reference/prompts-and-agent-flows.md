# Reference — Prompts & Agent Flows

Two ways an agent "does work": **prompts** (single-turn model calls) and **agent flows** (deterministic automation).

---

## Part 1 — Prompt tool (AI Builder prompts)

A **prompt** is a custom, single-turn instruction to an **Azure OpenAI / Azure AI Foundry** model. Add it at **agent level** (Tools → New tool → Prompt), **topic level** (Add node → Add a tool → New prompt), or as a **node in an agent flow** (Add an action → AI capabilities → Run a prompt).

### Prompt editor — what you configure
- **Instructions** — write manually, generate with Copilot from a description, or start from a **prompt-library template**.
- **Model & settings** — chat model, **temperature** (randomness/creativity), knowledge-retrieval settings, include-links toggle, **code interpreter** and **reasoning** toggles.
- **Inputs** — text and images; plus **sample data** to test.
- **Knowledge** — process/include **Dataverse tables** (note: anonymous/no-auth agents **can't** use Dataverse tables as knowledge in a prompt — but can still set inputs, model, temperature).
- **Output formatting**.
- **Test** with sample data → generates a response.

### Prompt tiers & billing (matters for cost)
Prompt tools bill as **"Text and generative AI tools"** by model tier (see [licensing-and-credits.md](licensing-and-credits.md)):
- **Basic** — 1 credit / 10 responses (0.1 credit per 1K tokens).
- **Standard** — 15 credits / 10 responses (1.5 credit per 1K tokens).
- **Premium** — 100 credits / 10 responses (10 credits per 1K tokens); also the meter for **reasoning models**.

### Prompt best practices
Specific, example-driven, simple, **brief** (long prompts → latency/timeouts), give a "way out" ("respond 'not found' if absent"), test & refine. Region-limited; may be throttled. See [../guidance/instructions-prompting-and-descriptions.md](../guidance/instructions-prompting-and-descriptions.md).

---

## Part 2 — Agent flows

**Agent flows** are Copilot Studio's deterministic automation (the classic automation experience). Same input → same output. They **consume Copilot Studio capacity per action** (unlike Power Automate cloud flows, which use Power Automate licensing).

> New-experience equivalent is "workflows" (public preview). Editing an agent flow from the new experience opens the classic designer in a new tab.

### Structure: trigger + ≥1 action
**Triggers**: instant (on-demand), scheduled, event-based, or **"When an agent calls the flow"** (required to add the flow as a **tool** in an agent).

**Action types**:
- **AI capabilities** — generate text, process documents, **Run a prompt**, call an agent, create a natural-language reply to a calling agent.
- **Human in the loop** — approvals, request info.
- **Built-in tools** — loops, branching, data operations, date/time, child flows.
- **Connectors** — M365, third-party, custom.

### How agents call flows & capacity
- From a **topic**: consumes **1 Classic answer + the agent-flow actions**.
- Via **generative orchestration**: consumes **1 Agent action (5 credits) + the agent-flow actions**.
- Agent-flow actions bill at **13 credits per 100 actions**.
- **Test runs** (flow designer or agent test chat) **don't** consume capacity.
- **M365 Copilot-licensed users**: agent-flow actions are **no charge** *only* for the **"When an agent calls the flow"** trigger; other triggers bill normally.

### Agent-flow enforcement (distinct from agent overage)
When prepaid capacity is exhausted, **new agent-flow runs are blocked** (in-progress runs finish); the **parent agent keeps working** for non-flow interactions. Resets monthly. Resolve via reallocating capacity, buying credits, or pay-as-you-go. **Power Automate cloud flows are NOT subject to this** (they use Power Automate licensing). Set per-agent monthly caps in PPAC → Licensing → Copilot Studio → Manage Agents.

### Create / convert
- **Create**: Workflows page → **New agent flow**; build via **natural language** or **designer** (drag/drop actions, conditions, loops).
- **Convert a Power Automate cloud flow → agent flow**: flow must be in a **solution** and in the target environment; in Power Automate, **Edit → change plan to Copilot Studio → Save → confirm**. **One-way, irreversible** (billing changes). You can still access it from Power Automate.
- Agent flows live in **solutions** (drafts, versioning, export/import).

### On disk (VS Code clone)
Flow-backed tools appear as `workflows/<Name>/metadata.yaml` + `workflow.json`; the tool wrapper is under `actions/`. See [tools.md](tools.md).

### When to use which
- **Prompt** — model reasoning/generation/extraction on text/images; scenario-specific outputs.
- **Agent flow** — deterministic multi-step automation, system integration, approvals; when you don't want per-step reasoning.
- **Topic** — deterministic *conversation* flow.
- **Connector/REST/MCP tool** — direct single API/action calls.
