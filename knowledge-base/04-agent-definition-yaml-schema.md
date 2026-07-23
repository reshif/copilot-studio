# 04 — Agent Definition: Folder Layout & YAML Schema

When you **clone** an agent, the extension downloads the **full agent definition** (not just the solution file) into a structured local folder.

## Cloned folder layout

```text
my-agent/
├── actions/                      # Connectors / connector actions
│   ├── DevOpsAction.mcs.yml
│   └── GetItems.mcs.yml
├── knowledge/
│   └── files/                    # Knowledge sources / uploaded files
│       ├── source1.yaml
│       └── source2.yaml
├── topics/                       # Conversation topics (AdaptiveDialog)
│   ├── greeting.mcs.yaml
│   ├── help.mcs.yaml
│   └── escalate.mcs.yaml
├── workflows/                    # Agent tools/actions backed by Power Automate flows
│   ├── GetDevOpsItems/
│   │   ├── metadata.yaml
│   │   └── workflow.json
│   └── GetMeetings/
│       ├── metadata.yaml
│       └── workflow.json
├── trigger/                      # Event/external triggers
│   └── welcometrigger.mcs.yaml
├── agent.mcs.yaml                # MAIN agent definition
├── icon.png                      # Agent icon (test pane + supported channels)
├── settings.mcs.yml              # Agent configuration settings
└── connectionreferences.mcs.yml  # Connection references used by connectors/actions
```

> Note: docs show both `.mcs.yaml` and `.mcs.yml` in examples — treat them as the same Copilot Studio YAML format. Some tools/framework docs also use type-suffixed names like `*.topic.mcs.yml`, `*.action.mcs.yml`, `*.knowledge.mcs.yml`.

## The `.mcs.yml` format

- `.mcs.yml` signals a file follows the **Copilot Studio YAML authoring schema** (MCS = Microsoft Copilot Studio).
- Every top-level file has a **`kind:`** property that determines which schema validates it.
- The authoritative schema lives in `reference/bot.schema.yaml-authoring.json` (JSON Schema; uses `$ref` chains). IntelliSense in the extension is driven by this schema.

## File → `kind` map

| Location | File pattern | `kind` | Purpose |
|----------|--------------|--------|---------|
| root | `agent.mcs.yml` | `GptComponentMetadata` | Agent metadata & top-level config (instructions, grounding). |
| root | `settings.mcs.yml` | `GptComponentSettings` | Agent instructions/settings. |
| `topics/` | `*.topic.mcs.yml` | `AdaptiveDialog` | Conversational topics with triggers. |
| `actions/` | `*.action.mcs.yml` | `TaskDialog` | Connector actions or connected-agent actions. |
| `knowledge/` | `*.knowledge.mcs.yml` | `KnowledgeSourceConfiguration` | Knowledge source definitions. |
| `adaptive-cards/` | `*.json` | (n/a) | Adaptive Card templates. |
| `agents/<Name>/` | `agent.mcs.yml` | `AgentDialog` | Child agent definitions. |

## Valid `kind` values (by category)

- **Agent-level**: `GptComponentMetadata`, `GptComponentSettings`, `AgentDialog`.
- **Dialog types**: `AdaptiveDialog` (topics), `TaskDialog` (actions).
- **Triggers** (used as `beginDialog.kind` in a topic, or in `trigger/`): `OnConversationStart`, `OnRecognizedIntent`, `OnActivity`, `OnUnknownIntent`, `OnToolSelected`, `OnRedirect`, plus external: `ExternalTriggerConfiguration` / `WorkflowExternalTrigger`.
- **Actions / nodes** (inside `beginDialog.actions`): `SendActivity`, `Question`, `BeginDialog`, `SetVariable`, `ConditionGroup`, `SearchAndSummarizeContent`, `AnswerQuestionWithAI`, `EndDialog`, `HttpRequest` (HTTP node), and more.
- **Task inputs** (for actions/tools): `AutomaticTaskInput`, `ManualTaskInput`.
- **Knowledge sources**: `PublicSiteSearchSource`, `SharePointSearchSource`, `GraphConnectorSearchSource` (and custom API sources).

## Key schema notes per kind

**`GptComponentMetadata`** (root `agent.mcs.yml`)
- `instructions` — the agent's system prompt.
- `grounding.GenerativeActionsEnabled` — enables **semantic/generative routing** so the orchestrator can pick topics/tools via their **`modelDescription`** / descriptions.

**`AdaptiveDialog`** (topics)
- `beginDialog.kind` — the **trigger** type.
- `beginDialog.id` — required id.
- `beginDialog.actions` — the ordered list of node actions.
- `inputs` / `inputType` — automatic parameter collection when generative actions are enabled.
- `outputType` — data returned to the orchestrator for grounded response generation.

**`KnowledgeSourceConfiguration`** (knowledge sources)
- Line-1 comment `# Name:` provides a display name.
- Line-2 comment provides a **description** the orchestrator uses when there are more than the 25-source limit to choose among.

## Validation rules (what breaks a build)

- **Unique node/`id` values** across a topic — duplicate ids are a common error (watch out when copy/pasting nodes).
- **Power Fx expressions must be prefixed with `=`** (e.g. `condition: =!IsBlank(Topic.Answer)`).
- **No placeholders** left in (`_REPLACE`, `TODO`).
- Every `kind` must exist in `bot.schema.yaml-authoring.json` (no invented/hallucinated kinds).
- Required fields present per kind; correct indentation (YAML is whitespace-sensitive).

## Naming conventions (recommended)

**Files** — kebab-case, descriptive, type-suffixed:
- `create-ticket.tool.yaml`, `product-pricing-faq.yaml` (not `faq.yaml`), `.topic.yaml` / `.tool.yaml` / `.trigger.yaml` suffixes.

**IDs & variables** — camelCase, descriptive, no abbreviations:
- `userOrderNumber`, `productDetails`, `checkPaymentStatus` (not `check1`), `customerEmail` (not `custEmail`).

**Comments** — explain non-obvious logic:
```yaml
nodes:
  # Check if user is within business hours and eligible for live support
  # Business hours: 9 AM - 5 PM EST, Mon-Fri; Eligibility: Premium tier only
  - id: check-live-support-availability
    type: condition
```

## Editing aids in the extension

- `Ctrl+Space` — context-aware IntelliSense based on current node level.
- `Ctrl+F` — search variables/values across the whole agent.
- **Problems** pane (`Ctrl+Shift+M`) + red underlines — validation errors/warnings; click to jump.
- Changed/saved files highlighted in a different color.

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
