# 06 — Tools & Actions

**Tools** define the actions an agent can perform. In the Copilot Studio UI they appear in the agent's **Tools** area; in the cloned definition they live under the **`actions/`** folder (and related folders like `workflows/` and `trigger/` with extra metadata/JSON).

## Tool types

A tool can be any of:

- **Prompts** — reusable prompt "skills" (structured generative behavior).
- **Workflows** — **Power Automate flows** (the most common way to do real work: query systems, write records, send email).
- **CUA tools** — Computer-Use Agent tools.
- **Custom connectors** — your own connector definitions.
- **REST APIs** — direct API calls.
- **MCP connectors** — Model Context Protocol servers exposed as tools.

## How tools appear on disk

- `actions/*.action.mcs.yml` — connector actions / connected-agent actions. `kind: TaskDialog`.
- `workflows/<Name>/metadata.yaml` + `workflows/<Name>/workflow.json` — a Power Automate flow tool: the `metadata.yaml` describes it to Copilot Studio; the `workflow.json` is the flow definition.
- `trigger/*.mcs.yaml` — triggers, which typically **reference a workflow** (see below).
- `connectionreferences.mcs.yml` — the **connection references** connectors/actions depend on (auth bindings resolved per environment).

## Inputs & outputs

Action/tool inputs come in two kinds:

- **`AutomaticTaskInput`** — the orchestrator fills the value automatically from context/conversation (great for generative orchestration).
- **`ManualTaskInput`** — the value is supplied explicitly (from a variable or literal).

Outputs are returned to the agent and can be mapped into topic variables for later use.

## Triggers for tools/autonomous behavior

Triggers define **when** topics or actions activate — schedules, events, or conditional types. A trigger typically **references a workflow**:

```yaml
kind: ExternalTriggerConfiguration
externalTriggerSource:
  kind: WorkflowExternalTrigger
```

This is the backbone of **autonomous agents**: an external/workflow event fires → the agent's orchestration decides how to respond using its tools.

## How orchestration picks a tool

Under generative orchestration (enabled via `grounding.GenerativeActionsEnabled`), the model chooses which tool to call based on the tool's **name and description**. Therefore:

- Give tools **clear, distinct names and descriptions**.
- Watch the **30–40 choice** rule of thumb — when a single agent has more than ~30–40 combined tools/topics/agents, routing precision degrades. Split into connected agents (see [09](09-multi-agent-orchestration.md)) or tighten descriptions.

## Connection references & auth

- Connectors and actions authenticate through **connection references** (`connectionreferences.mcs.yml`).
- Connection references are **environment-specific bindings** — when promoting an agent to another environment, connection references must be resolved/reconnected there (this is a classic ALM gotcha; solutions handle this).

## Practical patterns

- **"Do something in a system of record"** → Power Automate flow tool (`workflows/`), inputs as `AutomaticTaskInput` where possible.
- **"Call a simple REST endpoint"** → REST API tool, or an inline `HttpRequest` node inside a topic for one-off calls.
- **"Expose external capabilities"** → MCP connector or custom connector.
- **"Deterministic multi-step business process"** → author it as a **topic** that calls the tool, rather than relying purely on orchestration.

## Editing tools code-first

You can use GitHub Copilot / Claude Code (or hand-edit) to:
- Add a new action YAML under `actions/`.
- Adjust a tool's inputs/outputs and connection mode.
- Update a workflow's `metadata.yaml`.

Then **Apply** (see [03](03-vscode-extension.md)) and test in the Copilot Studio test pane. Remember Apply is live-but-not-published.

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
