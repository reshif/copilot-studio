# 05 — Topics & Dialogs (AdaptiveDialog)

A **topic** is a scripted conversational flow. In YAML it's a file of `kind: AdaptiveDialog`. Topics are Copilot Studio's deterministic building block — you use them for flows that must happen a specific way (compliance, escalation, structured data collection), while letting the orchestrator/generative answers handle open-ended Q&A.

> You never *have* to write YAML from scratch — Copilot Studio generates it as you build on the canvas. But code-first (VS Code) lets you author/clone/paste nodes fast, keep IDs unique, and use AI assistants.

## Anatomy of an AdaptiveDialog

Required: a **`beginDialog`** with a `kind` (the trigger) and an `id`, containing an ordered list of **`actions`** (nodes).

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart      # the trigger
  id: main
  actions:
    - kind: SendActivity
      id: sendMessage_M0LuhV
      activity:
        text:
          - Hello, I'm {System.Bot.Name}. How can I help?
        speak:
          - Hello and thank you for calling {System.Bot.Name}.
```

## Trigger kinds (`beginDialog.kind`)

| Trigger kind | Fires when |
|--------------|-----------|
| `OnConversationStart` | A conversation begins (greeting/welcome). |
| `OnRecognizedIntent` | User utterance matches the topic's trigger phrases / intent. |
| `OnActivity` | A specific activity type occurs. |
| `OnUnknownIntent` | No topic matched (fallback; e.g., conversational boosting / generative answers). Often `priority: -1`. |
| `OnToolSelected` | A tool is selected by orchestration. |
| `OnRedirect` | Reached via redirect from another topic. |
| `ExternalTriggerConfiguration` (`WorkflowExternalTrigger`) | External/workflow event (autonomous triggers). |

## Node/action kinds (inside `actions:`)

| Node `kind` | What it does |
|-------------|--------------|
| `SendActivity` | Sends a message (text/speak, Adaptive Cards). |
| `Question` | Asks a question, identifies an entity, stores the answer in a variable. |
| `ConditionGroup` | Branching via conditions (Power Fx expressions). |
| `SetVariable` | Sets a variable value (often via Power Fx). |
| `BeginDialog` | Calls/redirects into another dialog/topic. |
| `EndDialog` | Ends the current topic (optionally `clearTopicQueue: true`). |
| `SearchAndSummarizeContent` | **Generative answers via RAG** over knowledge sources. |
| `AnswerQuestionWithAI` | General-knowledge model answer. |
| `HttpRequest` | HTTP node (call a REST endpoint inline). |

## Example: generative answers ("conversational boosting") topic

This is the system fallback pattern — RAG over knowledge, then end if an answer was found:

```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  priority: -1
  actions:
    - kind: SearchAndSummarizeContent
      id: search-content
      userInput: =System.Activity.Text
      variable: Topic.Answer
      moderationLevel: Medium
      additionalInstructions: Include emojis to make responses more fun.
      publicDataSource:
        sites:
          - "www.chessusa.com/"
          - "www.chess.com/"
          - "www.lichess.org/"
      sharePointSearchDataSource: {}

    - kind: ConditionGroup
      id: has-answer-conditions
      conditions:
        - id: has-answer
          condition: =!IsBlank(Topic.Answer)
          actions:
            - kind: EndDialog
              id: end-topic
              clearTopicQueue: true
```

## Question node (with entity + variable)

```yaml
- kind: Question
  id: question_1
  alwaysPrompt: true
  variable: init:Topic.Continue
  prompt: Can I help with anything else?
  entity: BooleanPrebuiltEntity
```

- `variable: init:Topic.Continue` — declares/initializes a **topic-scoped** variable.
- `entity: BooleanPrebuiltEntity` — uses a prebuilt entity to interpret the answer. Other prebuilt entities exist (Person name, Date and time, etc.); you can also define custom entities and use **slot filling**.
- `conversationOutcome: ResolvedImplied` — you can tag outcomes for analytics.

## Condition node with Power Fx

```yaml
- kind: ConditionGroup
  id: condition-1
  conditions:
    - id: condition-1-item-0
      condition: =Topic.Continue = true
      actions:
        - kind: SendActivity
          id: sendMessage_4eOE6h
          activity: Go ahead. I'm listening.
```

All `condition:` values are **Power Fx**, so they start with `=`. See [08-power-fx-and-variables.md](08-power-fx-and-variables.md).

## Advanced things topics can contain

- **Entities & slot filling** (prebuilt + custom).
- **Variables** (Topic/Global/System/Environment scopes).
- **Conditions** via Power Fx.
- **HTTP nodes** to call REST APIs inline.
- **Adaptive Cards** (rich UI, forms, confirmations) — templates live in `adaptive-cards/*.json`; "Ask with Adaptive Card" collects structured input.
- **Redirects** to other topics or to child/connected agents.

## Topic inputs & outputs

Topics can declare **input** and **output** variables so data passes between topics when one redirects to another. Under generative orchestration, `inputs`/`inputType` allow **automatic parameter collection**, and `outputType` returns data to the orchestrator for grounded responses.

## Authoring cautions (from Microsoft)

- Errors in punctuation/syntax can break the conversation or produce cryptic errors; **support can't remediate code-editor errors** — copy a topic before heavy edits.
- When cloning/pasting nodes, **make every `id` and variable unique**.
- Keep `modelDescription` / trigger phrases high quality — orchestration routes to topics based on them.

## When to use topics vs generative

| Use a **topic** when… | Lean on **generative/orchestration** when… |
|-----------------------|--------------------------------------------|
| The flow must be deterministic (KYC, escalation, payment). | Open-ended Q&A over knowledge. |
| You need exact wording / compliance. | You want the model to choose tools/knowledge dynamically. |
| Structured multi-step data collection. | Rapidly changing content better served by RAG. |

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
