# Reference — Triggers (complete)

A **trigger** determines when a topic runs. In YAML it's the `beginDialog.kind` of an `AdaptiveDialog`. Default is **The agent chooses** (generative) or **User says a phrase** (classic).

## Trigger types

| Trigger (UI) | Fires when | Notes / YAML |
|--------------|-----------|--------------|
| **The agent chooses** | Generative agent decides topic name+description match the message. | Generative only. Write the **description** here. |
| **User says a phrase** | One+ trigger phrases match. | Classic only. 5–10 phrases recommended; up to **200/topic**; upload/download supported (≤3 MB file). `OnRecognizedIntent` with `intent.triggerQueries`. |
| **A message is received** | Any message activity (user types/says something). | `OnActivity`-family; most common. |
| **A custom client event occurs** | An event activity received. | Filter with **Event name** property. |
| **An activity occurs** | Any activity type. | Optional **Activity type** filter. |
| **The conversation changes** | Conversation-update activity (e.g. Teams user joins). | |
| **It's invoked** | Invoke activity (mostly Teams message/search extensions). | |
| **It's redirected to** | Topic explicitly called by another topic. | `OnRedirect`. |
| **The user is inactive for a while** | No interaction after configured time. | `InactivityTimer.*` system vars. |
| **A plan completes** | Generative agent finished all planned steps. | Generative only. |
| **An AI-generated response is about to be sent** | Generative agent generated a response. | Inspect `Response.FormattedText`; set `ContinueResponse=false` to suppress and send your own. |
| **On Error** | An error occurs. | `Error.Code`, `Error.Message`. HTTP raise-error routes here. |
| **On Sign In** | Sign-in needed. | `SignInReason`. |
| **On Conversation Start** | Conversation begins. | `OnConversationStart` — greetings. |
| **On Unknown Intent** | No topic matched. | `OnUnknownIntent`, often `priority: -1`; `FallbackCount`. Conversational boosting. |
| **On Select Intent** | Recognizer needs disambiguation. | `Recognizer.*` system vars. |
| **External / Workflow trigger** | External event via a workflow. | `ExternalTriggerConfiguration` / `WorkflowExternalTrigger` — backbone of **autonomous** agents. |

## Trigger priority (order of execution)
When multiple triggers qualify for one activity:
1. **An activity occurs**
2. **A message is received** / **A custom client event occurs** / **The conversation changes** / **It's invoked**
3. **The agent chooses** / **User says a phrase**

Same-type ties → oldest first (creation order). Override with the **Priority** property.

## Trigger conditions
Each trigger can have a **condition** that must be true to fire (e.g. only when `channel = Microsoft Teams`). For complex logic switch **Builder → Formula** (Power Fx).

## Changing a trigger
Topics page → hover the **Trigger** node → **Change trigger** icon → pick type → **Edit** for properties (Activity type, Condition, Priority). For **The agent chooses**, write the description; for **User says a phrase**, enter phrases.

## Autonomous agents
An autonomous agent responds to **events** instead of (or in addition to) user turns:
- Use an **External/Workflow trigger** (`WorkflowExternalTrigger`) — a Power Automate flow / connector event fires the agent, which then uses generative orchestration + tools to act.
- On disk (VS Code clone), triggers live in `trigger/*.mcs.yaml` and typically **reference a workflow** (`workflows/<Name>/`).
- Combine with **A plan completes** / **AI response generated** triggers to post-process autonomous runs.

## Build guidance
- Greetings → **On Conversation Start**; fallback/RAG → **On Unknown Intent**; deterministic intents → **User says a phrase** (classic) or good **descriptions** (generative).
- Autonomous workflows → **External/Workflow trigger**; use **trigger conditions** to scope by channel/state.
- Use **Priority** deliberately when several triggers can match.
