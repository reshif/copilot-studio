# 08 — Power Fx & Variables

**Power Fx** is the low-code, Excel-like expression language used throughout Copilot Studio to compute values, parse strings, and evaluate conditions. In YAML, any Power Fx expression is **prefixed with `=`**.

## Variable scopes (prefixes)

To reference a variable in Power Fx, prefix it with its scope:

| Scope | Prefix | Meaning |
|-------|--------|---------|
| **System** | `System.` | Built-in variables available everywhere (e.g. `System.Activity.Text`, `System.Conversation.Id`, `System.Bot.Name`). |
| **Global** | `Global.` | Persist across the whole conversation; can be marked AI-visible. Set via `add-global-variable` in code-first tooling. |
| **Topic** | `Topic.` | Scoped to the current topic. |
| **Environment** | `Environment.` | Environment variables (config that differs per environment). |

Example: the system variable `Conversation.Id` is referenced as `System.Conversation.Id`.

Declaring/initializing a topic variable in a node: `variable: init:Topic.Continue`.

## Topic input/output variables

Each topic variable can be configured to **receive** its value from other topics, **return** its value to other topics, or both (Variables panel in the UI; declared in YAML). This is how data flows between topics on redirect.

## Literal value formats

| Type | Examples |
|------|----------|
| String | `"hi"`, `"hello world!"` |
| Boolean | `true`, `false` (only) |
| Number | `1`, `532`, `5.258`, `-9201` |
| Record / Table | `[1]`, `[45, 8, 2]`, `["cats","dogs"]`, `{ id: 1 }`, `{ name: "John", info: { age: 25 } }` |
| DateTime | `Time(5,0,23)`, `Date(2022,5,24)`, `DateTimeValue("May 10, 2022 5:00:00 PM")` |
| Blank | `Blank()` |
| Choice | *not supported* |

> **US-style numbering**: decimal separator is a dot (`12,567.892` = ~12.5k). Therefore **commas are parameter separators** in function calls.

## Common functions by type

- **String**: `Text`, `Concat`/`Concatenate`, `Len`, `Lower`/`Upper`/`Proper`, `IsMatch`/`Match`/`MatchAll`, `StartsWith`/`EndsWith`, `Find`, `Replace`/`Substitute`.
- **Boolean/logic**: `Boolean`, `And`/`Or`/`Not`, `If`/`Switch`.
- **Number**: `Decimal`/`Float`/`Value`, `Int`/`Round`/`RoundDown`/`RoundUp`/`Trunc`.
- **Record/Table**: `Count`/`CountA`/`CountIf`/`CountRows`, `ForAll`, `First`/`FirstN`/`Index`/`Last`/`LastN`, `Filter`/`Search`/`LookUp`, `JSON`, `ParseJSON`.
- **DateTime**: `Date`/`Time`, `DateValue`/`TimeValue`/`DateTimeValue`, `Day`/`Month`/`Year`/`Hour`/`Minute`/`Second`/`Weekday`, `Now`/`Today`/`UTCNow`, `DateAdd`/`DateDiff`/`TimeZoneOffset`.
- **Blank/error**: `Blank`/`Coalesce`/`IsBlank`/`IsEmpty`, `Error`/`IfError`/`IsError`/`IsBlankOrError`.

## Worked examples

**Set a variable (uppercase a name):**
```
Upper(Text(Topic.customerName))
```
Store in `Topic.capsName`, then use it in a message.

**Condition (14-day discount check):**
```
Topic.bookingDate > (DateAdd(Now(), 14))
```

**In YAML (condition prefixed with `=`):**
```yaml
- kind: ConditionGroup
  id: condition-1
  conditions:
    - id: has-answer
      condition: =!IsBlank(Topic.Answer)
      actions:
        - kind: EndDialog
          id: end
          clearTopicQueue: true
```

## Why Power Fx matters for our framework

- It's the **computational backbone**: functional model, explicit scoping (`Topic`/`Global`/`System`/`Environment`), rich string/table/date libraries.
- Every condition, `SetVariable`, and dynamic value routes through it — so validation of `=`-prefixed expressions is a key part of code-first authoring (the `validate` skill checks this).

Reference: Power Platform Power Fx **formula reference for Copilot Studio**. See [12-glossary-and-sources.md](12-glossary-and-sources.md).
