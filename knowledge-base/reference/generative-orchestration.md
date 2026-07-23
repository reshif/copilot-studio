# Reference — Generative Orchestration (complete)

**Orchestration** = how the agent decides what to do with each user message or event trigger. Two modes. New agents default to **generative**.

## Generative vs classic — behavior comparison

| Behavior | Generative orchestration | Classic orchestration |
|----------|--------------------------|-----------------------|
| Topics | Selected by their **description**. | Selected by **trigger-phrase** match. |
| Child/connected agents | Selected by description. | N/A. |
| Tools | Agent can call tools by **name+description**. | Tools only called explicitly from a topic. |
| Knowledge | Agent proactively searches knowledge. | Knowledge is a fallback (or explicit generative-answers node). |
| Multiple sources | Can combine topics + tools + knowledge in one turn; chains them in sequence. | Tries to pick a **single** topic; falls back to knowledge. |
| Asking for input | **Auto-generates questions** to fill missing topic/tool inputs. | You must author Question nodes. |
| Responding | **Auto-generates** a summarized response. | You author Message nodes (or call a tool). |

Toggle: **Settings → Generative AI → Orchestration → "Use generative AI orchestration for your agent's responses?"** Yes/No. Admins can disable generative orchestration environment-wide (then only classic is possible). Prebuilt-agent origin determines the initial mode.

## How generative orchestration works
1. **Selection.** On each user message/event, the agent picks one or more tools/topics/agents/knowledge sources. The dominant signal is the **description**; also name, input/output parameters and their names/descriptions.
2. **Multi-intent + chaining.** It can select several items and **call them in sequence**, generating any questions needed for missing inputs.
3. **Input filling.** Inputs default to **Dynamically fill with AI** — extracted from message/context, else the agent asks. Can override with a **Custom value** (literal/variable/Power Fx).
4. **Response.** It summarizes results from everything it used into one answer.
5. **Context.** Uses **recent conversation history** — so the same query can yield different responses in a fresh vs ongoing conversation (this is expected; enables follow-ups and avoids re-asking).

## Authoring descriptions (the single most important skill)
When generative orchestration is on, provide a **high-quality description** for every topic, tool, knowledge source, child/connected agent. On topics, the Trigger node shows **"The agent chooses"** where you write the description. Turning on generative mode for a classic agent auto-generates topic descriptions from trigger phrases (revise them).

**Writing style:** simple, direct, active voice, present tense ("This tool provides weather information"), bullet lists for series.

**Relevance:** include intent keywords; 1–2 sentence summary of what it does + benefit; a **unique, specific name** (a short phrase). State what it **can't** do to disambiguate from similar items.

**Good example (disambiguated pair):**
- *Current Weather* — "Provides the current weather for any location worldwide, including temperature and if it's raining/snowing. It doesn't get forecasts for future days."
- *Weather Forecast for Tomorrow* — "Provides next-day weather for any location worldwide, including temperature. It doesn't get the current weather for today."

**Anti-patterns:** vague ("Answer Question — This tool can answer questions."); jargon ("Get EPS" — spell out earnings per share).

> If multiple topics have **similar descriptions**, the agent may invoke all of them — test and differentiate.

## Control nodes / triggers for generative agents
- **End all topics** node — cancel remaining planned steps for the turn.
- **A plan completes** trigger — fires when all planned steps finished.
- **An AI-generated response is about to be sent** trigger — inspect `Response.FormattedText`; set `ContinueResponse = false` to suppress the auto response and send your own Message node.
- **Clear variable values → Conversation history for the current session** — reset planner context. (Reset Conversation system topic clears only global vars, **not** history.)

## Testing
Use the **activity map** (real-time) in the test panel to see which topics/tools/knowledge/agents handled a turn and how inputs were filled. Essential for debugging routing/disambiguation.

## Model & reasoning
Choose the model powering reasoning under agent settings (e.g. GPT-family; GPT-5 Chat GA in Copilot Studio US+EU from 2025-11-24). The **new agent experience** uses an enhanced orchestration runtime with **deep reasoning**, not toggleable, especially strong over Microsoft 365 data (see [../02-agent-experience-new-vs-classic.md](../02-agent-experience-new-vs-classic.md)).

## Multilingual
Generated content is in the active language (primary or a configured secondary); the agent detects user language from client/browser. Check language support for generative answers/orchestration.

## Known limitations (generative orchestration)
- **Knowledge:** doesn't use the **Conversational boosting** system topic for knowledge search — customizations there (and classic data sources configured in generative-answers nodes) are ignored. See [knowledge.md](knowledge.md#search-modes).
- **Custom entities as inputs:** tools/topics don't support closed-list/regex entities as **input parameters** — collect via a Question node instead.
- **Disambiguation:** doesn't call the **Multiple Topics Matched** system topic — may fail to disambiguate very similar topics; consider turning that system topic off during testing.
- **Conversation context is limited** — very old turns may be dropped; sometimes re-collect key info or restate it periodically.
- **Hyperlinks** from knowledge (Word/PDF/web) render as **plain text** in responses.

## Tool/agent count guidance
- Orchestrator hard max: **128 tools** per agent; **recommended ≤ 25–30** for quality. Child agents each get their own orchestration and up to 128 tools.
- Combined routable choices (tools+topics+agents) start degrading past **~30–40** — tighten descriptions or split into connected agents (see [../09-multi-agent-orchestration.md](../09-multi-agent-orchestration.md)).
