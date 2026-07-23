# Reference — Variables & Entities (complete)

## Part 1 — Variables

### Base types
Set on first assignment; **fixed thereafter** (assigning a different type errors). Untyped/unassigned shows as **unknown** in test.

| Type | Description |
|------|-------------|
| String | Text. |
| Boolean | `true` / `false` only. |
| Number | Any real number. |
| Table | List where all values share one type. |
| Record | Name–value pairs; values can be any type. |
| DateTime | Date/time/day/month relative to a point in time. |
| Choice | List of string values with synonyms. |
| Blank | Placeholder for "no/unknown value" (`Blank()`). |

Evaluation order: top→bottom of canvas; Condition branches left→right.

### Scopes (prefixes)
| Scope | Prefix | Notes |
|-------|--------|-------|
| Topic | `Topic.` | Default; usable only in the creating topic. Can be promoted to Global. |
| Global | `Global.` | Usable in all topics. Persists for the session. Reset by **Start over**. |
| System | `System.` | Auto-created, read-only, everywhere. |
| Environment | `Environment.` | Power Platform env variables; **read-only** in Copilot Studio. |

### Topic input/output
Each topic variable can be set to **receive** from other topics, **return** to other topics, or both. Under generative orchestration, `inputs`/`inputType` allow **automatic** input filling from context; `outputType` returns data to the orchestrator (see [generative-orchestration.md](generative-orchestration.md)).

### Environment variables (ALM)
Power Platform concept for values that differ per environment. **Read-only** in Copilot Studio (admins edit in Power Apps). Republish agents after changing an env variable — **except** type *secret*, which is retrieved at runtime. Type mapping:

| Copilot Studio type | Power Apps type |
|---------------------|-----------------|
| Decimal number | Number |
| JSON | Detected from value (non-JSON ⇒ Unspecified/validation error) |
| Text | String |
| Yes/No | Boolean |
| Data source | String |
| Secret | String |

### System variables (full table)
Reference with `System.` prefix in Power Fx (some are hidden in the picker and only reachable via formula).

| Name | Type | Definition |
|------|------|-----------|
| Activity.Attachments | table | File attachments the user provides. |
| Activity.Channel | choice | Channel ID of the conversation. |
| Activity.ChannelData | any | Channel-specific content object. |
| Activity.ChannelId | string | Channel ID as string. |
| Activity.From.Id | string | Channel-specific unique sender ID. |
| Activity.From.Name | string | Sender friendly name. |
| Activity.Name | string | Event name. |
| Activity.Recipient.Id | string | Incoming activity Type property. |
| Activity.Recipient.Name | string | Agent display name in channel (telephony: the phone number). |
| Activity.Text | string | Most recent user message. |
| Activity.Type | choice | Activity type. |
| Activity.TypeId | string | Activity type as string. |
| Activity.Value | any | Open-ended value. |
| Bot.EnvironmentId | string | Environment ID. |
| Bot.Id | string | Agent ID. |
| Bot.Name | string | Agent name. |
| Bot.SchemaName | string | Agent schema name. |
| Bot.TenantId | string | Tenant ID. |
| ClientPluginActions | choice | Dynamic client plugin actions for generative orchestration. |
| Conversation.Id | string | Unique conversation ID. |
| Conversation.InTestMode | Boolean | True if in the test canvas. |
| Conversation.LocalTimeZone | string | IANA time zone name. |
| Conversation.LocalTimeZoneOffset | datetime | UTC offset for local time. |
| Error.Code | string | Current error code (On Error trigger). |
| Error.Message | string | Current error message (On Error trigger). |
| FallbackCount | number | Times a topic couldn't be matched (On Unknown Intent). |
| InactivityTimer.Continue | Boolean | Whether inactivity timer continues (Inactivity trigger). |
| InactivityTimer.Count | number | Inactivity fire count (Inactivity trigger). |
| LastMessage.Id | string | ID of previous user message. |
| LastMessage.Text | string | Previous user message. |
| Recognizer.ExtractedEntities | choice | Extracted entities (On Select Intent). |
| Recognizer.IntentOptions | choice | Intent options when ambiguous (On Select Intent). |
| Recognizer.SelectedIntent | choice | Selected intent (On Select Intent). |
| Recognizer.TriggeringMessage.Id | string | ID of message that triggered topic. |
| Recognizer.TriggeringMessage.Text | string | Message that triggered topic. |
| Recognizer.MultipleTopicsMatchedReason | string | Why multiple topics matched (On Select Intent). |
| SignInReason | choice | Which sign-in option is needed (On Sign In). |
| User.Language | choice | User language locale per conversation. |

**Voice-enabled agents:** `Activity.InputDTMFKeys`, `Activity.SpeechRecognition.Confidence`, `Activity.SpeechRecognition.MinimallyFormattedText`, `Activity.UserInputType`, `Conversation.OnlyAllowDTMF`, `Conversation.SipUuiHeaderValue`.

### Authentication variables
Availability depends on the auth mode (see [publishing-security-limits.md](publishing-security-limits.md)):

| Variable | No auth | Authenticate with Microsoft | Manual |
|----------|---------|-----------------------------|--------|
| User.DisplayName | — | ✓ | ✓ |
| User.FirstName / LastName | — | ✓ | ✓ |
| User.PrincipalName | — | ✓ | ✓ |
| User.Email | — | ✓ | ✓ |
| User.Id | — | ✓ | ✓ |
| User.IsLoggedIn | — | ✓ | ✓ |
| User.AccessToken | — | — | ✓ (manual only) |
| SignInReason | — | ✓ | ✓ |

`User.DisplayName` ← `name` claim (needs `profile` scope); `User.Id` ← `sub` claim. **Never** put `User.AccessToken` in Message nodes or untrusted flows. Test-pane overrides: `/debug set bot.UserDisplayName "X"`, `/debug set bot.UserID ""` (ID can only be blanked, not set).

---

## Part 2 — Entities

An **entity** is a unit of real-world information the NLU extracts from user input and slot-fills into a variable.

### Prebuilt entities → variable base type
| Entity | Base type |
|--------|-----------|
| Multiple-choice options | Choice |
| User's entire response | String |
| Age | Number |
| Boolean | Boolean |
| City / Color / Continent / Country or region | String |
| Date and time | DateTime |
| Email | String |
| Event | String |
| Integer | Integer |
| Language | String |
| Money | Number |
| Number | Number |
| Ordinal | Number |
| Organization | String |
| Percentage | Number |
| Person name | String |
| Phone number | String |
| Point of interest | String |
| Speed | Number |
| State | String |
| Street address | String |
| Temperature | Number |
| URL | String |
| Weight | Number |
| Zip code | String |
| Custom entity | Choice |

(Teams-plan agents map several of these to String; **Duration** is available there too.)

### Custom entities
Created under **Settings → Entities → Add an entity → New entity**. Two kinds:

**Closed list** — a defined list of values.
- Add **synonyms** per value (e.g. `hiking` → `trekking`, `mountaineering`).
- **Smart matching** — fuzzy logic: autocorrects misspellings, semantic expansion (e.g. "softball" → "baseball").
- Upload/replace via text file (≤3 MB), one value per line, `|` delimits synonyms:
  ```text
  hiking|trekking
  hiking|mountaineering
  cycling|bicycling
  ```

**Regular expression (regex)** — pattern matching (tracking IDs, license/credit-card numbers, IPs).
- NLU/CLU use **.NET regex**; NLU+ uses **JavaScript regex**.
- Case-sensitive by default; `(?i)` for case-insensitive; `|` to combine patterns.

**Open list / dynamic inline** — values populated at **runtime** from a table variable (Excel, Dataverse, Power Fx). ≤100 entries. For synonyms, use a list of records `{ DisplayName, Synonyms[] }`. Common for classic voice agents. Configure the Question node with **Identify → Options from a list variable**.

### Using entities in a Question node
- **Identify** dropdown: prebuilt/custom entity, Multiple choice options, User's entire response, Options from a list variable, or **One of multiple entities** (≤5 entities; one per type; no external entities; returns a *record* like `Identifier.account`/`Identifier.phone`/`Identifier.unknown`; recognizes only the *first* matched entity).
- **Include metadata** → `.Literal`, `.Value`, `.ConfidenceScore`.
- **Allow multiple values** → returns a *table*.

### Slot filling
Placing the extracted entity value into the variable at the Question node. **Proactive slot filling**: the agent listens continuously and fills slots from earlier user statements, **skipping** Question nodes whose slot is already filled (unless **Skip question → Ask every time**). Example: "I want to buy hiking boots under $100" fills product category **and** price range in one turn.

> **Generative orchestration limitation:** tools and topics **don't** support custom entities (closed-list/regex) as **input parameters** yet. To collect via a custom entity, use a **Question node** inside a topic.
