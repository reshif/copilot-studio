# Reference — Topic Node Types (complete)

A topic (`kind: AdaptiveDialog`) is a set of **nodes** (called `actions` in YAML) that run top-to-bottom, left-to-right through branches. This is the exhaustive node reference: what each does, key properties, and its YAML `kind`.

> **Naming note:** the canvas generates YAML using `SendMessage` for a message and `Question` for a question; some docs samples also show `SendActivity`. Both exist in the schema. When cloning via the VS Code extension, trust what the schema/IntelliSense shows for your version, and `validate` before Apply. Node names can be up to **500 characters**. You can't rename **Trigger** or **Go to step** nodes.

## Node catalog

| Canvas node | YAML `kind` | Purpose |
|-------------|-------------|---------|
| Send a message | `SendMessage` (a.k.a. `SendActivity`) | Send text/speech, formatting, variables, cards to the user. |
| Ask a question | `Question` | Ask, identify an entity, store answer in a variable. |
| Ask with adaptive card | (adaptive card question) | Interactive card with input fields/buttons. |
| Add a condition | `ConditionGroup` | Branch on Power Fx conditions; supports `elseActions`. |
| Set a variable value | `SetVariable` | Assign a value (literal, variable, or Power Fx). |
| Parse value | (parse) | Convert a value (e.g. JSON/`Any`) into a typed Power Fx record. |
| Clear variable values | (clear) | Clear variables, incl. **conversation history for the current session**. |
| Loop through a list | (loop / `Foreach`) | Iterate over a table/collection. |
| Redirect to another topic | `BeginDialog` | Call a subtopic; returns to caller when done. |
| Go to another topic (transfer) | (redirect) | Some system-topic redirects end the whole conversation. |
| End topic / End conversation | `EndDialog` | End topic (`clearTopicQueue: true`) or conversation (survey/handoff). |
| End all topics | (cancel plan) | Cancel remaining orchestrator-planned steps. |
| Call a tool / action | (action/`InvokeFlowAction` etc.) | Run a Power Automate flow, connector, prompt, REST, MCP tool. |
| Add generative answers | `SearchAndSummarizeContent` | RAG over knowledge sources. |
| Answer with general AI | `AnswerQuestionWithAI` | Model general-knowledge answer (no grounding). |
| Send HTTP request | (HTTP request) | Call an external REST API inline. |
| Authenticate | (authenticate) | Prompt user sign-in; leaf node only; adds success/failure paths. |
| Send an event / activity | (send event) | Emit an event/activity (e.g. client tool call). |
| Transfer to agent (handoff) | (handoff) | Escalate to a live agent (Omnichannel etc.). |

---

## Message node — `SendMessage`
Sends a message. Supports basic formatting (bold, italics, lists, hyperlinks), variables (`{x}`), and both `text` and `speak` (for voice).

```yaml
- kind: SendMessage
  id: Sjghab
  message: I am happy to help you place your order.
```
Rich variant with text + speak:
```yaml
- kind: SendActivity
  id: sendMessage_M0LuhV
  activity:
    text:
      - Hello, I'm {System.Bot.Name}. How can I help?
    speak:
      - Hello and thank you for calling {System.Bot.Name}.
```

**Generative-orchestration tip:** for flexible agents, don't send the final answer in a Message node — return it as a **topic output variable** so the orchestrator can compose a contextual response.

---

## Question node — `Question`
Asks a question, identifies an entity, and slot-fills a variable.

```yaml
- kind: Question
  id: eRH3BJ
  alwaysPrompt: false          # false => skip if slot already filled (proactive slot filling)
  variable: init:Topic.State   # init: declares/initializes the variable
  prompt: To what state will you be shipping?
  entity: StatePrebuiltEntity  # prebuilt or custom entity
```

Key properties/behaviors:
- **`entity`** — what to listen for. Use prebuilt entities (`BooleanPrebuiltEntity`, `StatePrebuiltEntity`, `Money`, `Date and time`, etc.), custom entities, "Options from a list variable" (dynamic inline), or "One of multiple entities" (up to 5).
- **`alwaysPrompt` / Skip question** — `false` (Ask only if needed) enables proactive slot filling; `true` (Ask every time) always prompts.
- **Multiple choice / options** — show entity values as buttons; each choice auto-creates a conditional path.
- **Allow multiple values** — returns a *table* of recognized values (loop with **Loop through a list**).
- **Include metadata** — exposes `Var.Literal` (raw words), `Var.Value` (typed value), `Var.ConfidenceScore` (0–1) — variable becomes *record*.
- **Reprompt / retry logic** — behavior when nothing recognized.
- **`conversationOutcome`** — tag outcomes (e.g. `ResolvedImplied`) for analytics.

---

## Condition node — `ConditionGroup`
Branches on **Power Fx** conditions (each starts with `=`). Supports `elseActions` for the "all other conditions" path.

```yaml
- kind: ConditionGroup
  id: sEzulE
  conditions:
    - id: pbR5LO
      condition: =Topic.State = "California" || Topic.State = "Washington" || Topic.State = "Oregon"
      actions:
        - kind: SendMessage
          id: a1
          message: Standard shipping applies.
  elseActions:
    - kind: SendMessage
      id: X7BFUC
      message: There will be an additional shipping charge of $27.50.
```
Branches evaluate **left to right**. Use `is not Blank` / `IsBlank()` patterns for "one of multiple entities" routing.

---

## Variable management nodes
- **Set a variable value** (`SetVariable`) — assign literal/variable/Power Fx: e.g. `capsName = Upper(Text(Topic.customerName))`.
- **Parse value** — convert `Any`/JSON into a typed Power Fx record (commonly used on HTTP error bodies or flow outputs).
- **Clear variable values** — clear specific variables, all topic vars, all global vars, or **Conversation history for the current session** (important for generative orchestration — resets planner context; note the **Reset Conversation** system topic does *not* clear conversation history, only global vars).

---

## Topic management nodes
- **Redirect to another topic** (`BeginDialog`) — runs a subtopic then returns; you can add nodes after the redirect. Pass **inputs**; **outputs** become topic variables.
- **Redirecting to these system topics ENDS the whole conversation:** End of Conversation, Confirmed Success, Confirmed Failure, Goodbye, Escalate, Start over (Start over also resets global variables).
- **End topic / End conversation** (`EndDialog`) — `clearTopicQueue: true` clears queued steps. On end-of-conversation you can add a **CSAT survey** (End with survey) or **Transfer to agent** (handoff).
- **End all topics** — cancels remaining orchestrator-planned steps for the turn.

---

## Add generative answers — `SearchAndSummarizeContent`
RAG over knowledge. See [knowledge.md](knowledge.md) for source config.
```yaml
- kind: SearchAndSummarizeContent
  id: search-content
  userInput: =System.Activity.Text
  variable: Topic.Answer
  moderationLevel: Medium
  additionalInstructions: Keep answers under 3 sentences.
  publicDataSource:
    sites:
      - "www.contoso.com/support/"
  sharePointSearchDataSource: {}
```
Follow with `=!IsBlank(Topic.Answer)` to branch on found/not-found. `AnswerQuestionWithAI` is the ungrounded general-knowledge equivalent.

---

## Send HTTP request node
Calls an external REST API inline.
- **Methods:** GET, POST, PATCH, PUT, DELETE.
- **Headers:** key/value pairs (e.g. `Authorization: Bearer <token>`).
- **Body:** No content / **JSON Content** (static or Power Fx object) / **Raw content** (Power Fx string).
- **Response data type:** provide sample JSON → **Get schema from sample JSON** generates a Power Fx-typed variable with IntelliSense. Store into a variable via **Save response as**.
- **Error handling:** **Raise an error** (default → triggers **On Error** system topic) or **Continue on error** → store `StatusCode` and `ErrorResponse` (type `Any`; parse with a **Parse value** node) and keep running.
- **Request timeout:** milliseconds; default **30 seconds**.

> Use HTTP node for one-off calls; for reusable, governed integrations prefer a **tool** (connector/flow/REST tool). See [tools.md](tools.md).

---

## Authenticate node
Prompts the user to sign in (manual auth). **Leaf-node only** — added at the end of a dialog path; then success/failure child paths appear. Populates `User.IsLoggedIn`, `User.AccessToken`, etc. See [publishing-security-limits.md](publishing-security-limits.md).

---

## System topics (built-in) you'll interact with
- **Conversation Start** — greeting; good place to disclose AI-generated content.
- **Conversational boosting** — holds the default generative-answers node (classic mode). *Not used by generative orchestration for knowledge search.*
- **Sign in** — manual-auth sign-in at conversation start.
- **On Error** — global error handler (HTTP raise-error triggers it; `Error.Code`, `Error.Message`).
- **Multiple Topics Matched** — disambiguation (Did-you-mean). *Generative orchestration doesn't call it — consider turning it off while testing.*
- **Reset Conversation / Start Over** — resets global vars (NOT conversation history by default).
- **Escalate, End of Conversation, Goodbye, Confirmed Success/Failure** — terminal system topics.

You can turn system topics on/off and edit them, but avoid editing until you're comfortable building full flows. You **can't create or delete** system topics.

## Authoring cautions
- Keep every node **`id` unique** (especially when pasting). Power Fx conditions must start with `=`.
- Avoid **periods (`.`) in topic names** — a solution containing an agent with a `.` in any topic name can't be exported.
- Designing very complex topics entirely in the code editor "isn't fully supported" per Microsoft — validate and test.
