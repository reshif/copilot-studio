# Reference — Tools (complete)

**Tools** let an agent interact with external systems (send email, read/write Dataverse, get weather, post to Teams…). With generative orchestration the agent picks tools automatically by name+description; in classic mode tools are only called explicitly from a topic.

## Tool mechanisms (types)
| Type | What it is |
|------|-----------|
| **Connector** (prebuilt) | Thousands of preset Power Platform connectors (Microsoft + third-party). |
| **Connector** (custom) | Your own connector definition; needs org **view & share** permissions. |
| **Agent flow** | A Power Automate–style flow with one or more actions. |
| **Prompt** | Single-turn, model-based prompt; can reference knowledge and generate code to analyze data. |
| **REST API** | Connect to a REST API; pick endpoints/methods as tools. |
| **Model Context Protocol (MCP)** | Connect to an MCP server to access its tools + resources. |
| **Computer use (CUA)** | Interact with GUIs of websites/desktop apps (click, menus, type). |
| **Skill** | Container for a set of related tools. |
| **Client tool** | Send an event activity to the client to perform an action and return a response. |

Add via **Tools page → Add a tool → New tool** (Prompt / Agent flow / Computer use / Custom connector / MCP / REST API) or add an existing one. In a topic, **Add node → Add a tool** (tabs: Basic tools / Connector / Tool).

## Tool configuration — three sections

### 1. Details
- **Name** — clear, function-indicating (shows in tools list).
- **Description** — orchestration relies on this to decide when to use the tool. Be specific: what it does + when to use it.
- **Allow agent to decide dynamically when to use the tool** — on ⇒ generative orchestration may call it; off ⇒ only explicit topic calls. (On by default when generative orchestration is enabled.)
- **Ask the end user before running** — confirmation prompt before execution (default No).
- **Authentication** — **End user** vs **Maker-provided** credentials (see below).

### 2. Inputs
Each input row has a **Fill using** mode:
- **Dynamically fill with AI** (default) — extract from context/message; else auto-generate a question. No manual Question nodes needed.
- **Custom value** — override with literal / variable / Power Fx (agent won't ask).

**Customize** an input for: Display name & Description, **Identify as** (string or entity), **Retry logic**, **Input validation** (beyond the entity default).

### 3. Completion — "After running"
- **Don't respond (default)** — agent folds the tool output into its generated response.
- **Write the response with generative AI** — AI crafts a contextual response from outputs.
- **Send specific response** — templated response with variable/Power Fx insertion.
- **Send an adaptive card** — rich interactive response.
- Also choose which **output variables** to expose to the agent/other tools.

## Tool selection factors (generative)
Name+description, current conversation context, user intent, available inputs/outputs, and previous tool usage in the conversation.

## Authentication for tools
Tools run in the agent runtime **in the user context** and require auth enabled (an agent set to **No authentication** can't use tools with user credentials).
- **End-user credentials** — uses the signed-in user's identity; user only accesses what they're authorized to. Best for per-user data.
- **Maker-provided credentials** — uses the author's credentials; for shared resources or when users shouldn't need individual access.

## MCP specifics
For MCP connectors the config page shows **Tools** and **Resources** sections (names/descriptions of what the server exposes) instead of Inputs/Completion.

## Enable / disable / delete
- **Enabled** toggle turns a tool off (stays connected, re-enableable).
- Delete only from the **agent's** Tools page (⋮ → Delete), not the global Tools page.

## Limits
- **128 tools** max per agent for the orchestrator; **recommended ≤ 25–30** for best results.
- **Skills:** 100 per agent.
- Child agents each have their own orchestration + up to 128 tools.

## On disk (VS Code clone)
- `actions/*.action.mcs.yml` (`kind: TaskDialog`) — connector/agent actions.
- `workflows/<Name>/metadata.yaml` + `workflow.json` — flow-backed tools.
- `connectionreferences.mcs.yml` — per-environment connection references (rebind on promotion).
- Task inputs use `AutomaticTaskInput` (AI-filled) or `ManualTaskInput` (explicit).

## Build guidance
- Prefer **flows/connectors/REST tools** (governed, reusable) over inline HTTP nodes for anything shared.
- One capability per tool; distinct, specific descriptions; keep total ≤ 25–30.
- Use **End-user** auth for user-scoped data, **Maker** auth for shared back ends.
- Turn on **Ask before running** for state-changing/irreversible actions.
