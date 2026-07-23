# Guidance — The Agent Design Playbook (what / when / where / how)

The decision playbook for building Copilot Studio agents that actually work in production. Synthesized from Microsoft's official **guidance library** (the enterprise-customer team's best practices), plus the reference material in this KB.

> Companion files: [instructions-prompting-and-descriptions.md](instructions-prompting-and-descriptions.md) (the writing craft) · [multi-agent-patterns.md](multi-agent-patterns.md) (splitting agents) · [field-notes-practitioner-tips.md](field-notes-practitioner-tips.md) (community wisdom).

---

## Step 0 — Do you need to design first, or just prototype?

Not every agent needs upfront design, but **skipping it causes rework, governance blockers, and misaligned outcomes.**

**Use a structured design framework if ANY of these are true:**
- The agent accesses **enterprise or sensitive data**.
- The agent **takes actions**, not just answers questions.
- **Multiple teams/stakeholders** are involved.
- **Security, compliance, or governance** requirements apply.
- It's expected to **scale, evolve, or be reused**.
- You're building an **autonomous agent**, a **multi-agent system**, or a **workflow/orchestration-heavy** agent.

**You can skip it for:** short-lived proofs of concept, a small static Q&A set, no tools/restricted data, pure learning.

**The winning approach: prototype first, design before scaling.** Design is most valuable at the transition from *"let's try this idea"* to *"let's make this reliable, secure, and scalable."*

Skipping design when you shouldn't produces: late-discovered governance problems, significant rebuilds, agents that technically work but miss business goals, and fragile solutions that can't scale.

---

## Step 1 — Choose the platform / agent type

Decide **before** building. Selection factors: **end-user needs** (employees vs customers; which channels), **integration requirements** (which systems, API availability, auth models), **scalability** (volume, process complexity), **security & compliance** (Zero Trust, Purview DLP, sensitivity labels).

| Agent type | Use when | Trade-off |
|------------|----------|-----------|
| **Declarative agent** (M365 Copilot / Agent Builder) | Extend M365 Copilot with domain-specific commands; low-code; governed inside M365 compliance boundary | Rides M365 Copilot's model/orchestrator; no Copilot Studio analytics |
| **Custom engine agent** (Copilot Studio) | Max flexibility: custom logic, external APIs, complex orchestration, multistep workflows, autonomy, multi-channel | You own orchestration, testing, governance |
| **Agents SDK** (pro-code) | Fully custom runtime/model; connect into Copilot Studio as an external agent | Full development ownership |

See [../reference/agent-types-and-m365-extensibility.md](../reference/agent-types-and-m365-extensibility.md).

---

## Step 2 — Choose the orchestration mode

**Generative orchestration** (default for new agents) uses an LLM planner that interprets intent, decomposes requests, selects tools/knowledge/topics/agents, and executes multistep plans.

It exists because classic topic-driven design causes: large topic inventories with overlapping logic, difficulty with ambiguous/multi-intent utterances, inconsistent handling of rephrasing, and high maintenance when APIs/rules change.

Generative fixes those by reducing topic sprawl, **automating slot filling**, adapting plan/response dynamically, improving relevance via semantic retrieval, and enabling proactive next-step suggestions.

**But choose deliberately:** the planner reasons **every turn**, so it costs more credits and adds latency. For **high-volume, predictable intents**, classic topic-trigger matching is cheaper and more deterministic. NLU models are great for specific intents but struggle with complex queries; generative handles a wider range but introduces latency.

> Generative orchestration is currently **English only**, and uses the **last 10 turns** of conversation history to fill inputs and pick capabilities.

---

## Step 3 — Define your three control layers (the key architectural move)

**Don't leave every decision to the AI.** Production agents have three layers:

| Layer | What it is | Use for |
|-------|-----------|---------|
| **Deterministic** | Rule-based, step-by-step, no AI interpretation. Either don't expose the action to the planner, or wrap it in a topic requiring confirmation. | Mission-critical / irreversible: payments, deletions, sensitive-data checks |
| **Hybrid (intercept)** | AI operates within set boundaries with human/rule interception checkpoints (approval steps, value limits, escalation thresholds) | Medium-risk processes — AI does the lifting, human oversees |
| **AI orchestrator** | Fully generative within guardrails and policy | Low-risk Q&A, lookups, simple multistep requests |

Then explicitly define **decision boundaries** — for every action/topic, decide whether it:
1. Can execute **without confirmation**,
2. Requires **in-conversation user confirmation** ("Are you sure you want to delete all records?"), or
3. Requires **offline approval** (admin approval workflow).

Enforce via confirmation nodes, the platform's **Ask the end user before running** tool setting, approval features, or trigger logic.

---

## Step 4 — The component decision matrix (what to use, when)

| Need | Use | Notes |
|------|-----|-------|
| Answer factual questions from content | **Knowledge (RAG)** | Not for document comparison or complex reasoning — see Step 5 |
| Deterministic conversation/compliance flow | **Topic** | Exact wording, structured collection, escalation |
| Do something in a system | **Tool** (connector / agent flow / REST) | Governed, reusable |
| Precise, highly structured model output | **AI prompt tool** | Full control of model, format, constraints |
| General reasoning + tool selection + light formatting | **Orchestrator** | Don't build a prompt for what the orchestrator does natively |
| Standardized, centrally-managed tool catalog across many agents | **MCP server** | Auto-discovery/versioning; update once, all agents follow |
| Quick prototype / one-off API | **Direct REST / connector / HTTP node** | Faster; skips MCP lifecycle overhead |
| Automate a GUI with no API | **Computer Use (CUA)** or **RPA** | See table below |
| Deterministic multi-step automation | **Agent flow** | Same input → same output |
| Complex multi-turn pro-code capability | **Skill** | Developer-built, client+server events |
| Separate domain, own governance/reuse | **Connected agent** | See multi-agent patterns |

### AI prompt vs orchestrator
- **Orchestrator**: fixed system prompt you can't edit (only influence via agent instructions); good for simple extraction ("get a name from text"), tool selection, light formatting.
- **AI prompt**: standalone, deeper model control — broader model choice (incl. Foundry), Dataverse grounding, file inputs, code interpreter. Use for **complex extraction** (multiple entities from a long report with domain relationships) or **strict stylistic/structural control**.

### MCP vs connector vs direct API
Use **MCP** when you need a standardized, centrally managed way to expose tools to **multiple agents** without per-client config, and especially when **upstream APIs change frequently** (update the server once; all agents follow without republishing). Use **direct API/connector** when no MCP server exists or you're **rapidly prototyping**.
*MCP limits:* you can't enrich tool descriptions with extra "when to invoke" context, and **topics can't call MCP servers directly**. Requires generative orchestration.

### RPA vs CUA
| Aspect | RPA | CUA |
|--------|-----|-----|
| Automation type | Rule based | LLM driven |
| Interaction | UI tree | Vision |
| Authoring | Script, complex | Natural language |
| Decisions | Predefined rules | Autonomous, visual |
| Flexibility | Limited | High |
| Error handling | Static | Self-correcting from visual feedback |

**Use RPA** when: GA-only features required, UI is stable, rules are clear, speed/high volume matters, an RPA team owns it.
**Use CUA** when: UIs shift/vary, you need it fast (RPA backlog full), the task depends on what's visible (charts/colors/dynamic layouts), decisions are fuzzy and need self-correction.
*Machines:* use **BYO machines for production** (Entra ID, Intune, web+desktop); **hosted machines only for prototyping** (shared pool, not Entra-joined, one Cloud PC per user, throttled).

---

## Step 5 — Build each component well

### Descriptions & names (the routing logic)
- **Names carry more weight than descriptions** — a tool named `TranslateText` beats `Flow1`. Avoid cryptic names. If the agent picks the wrong thing, revisit names first.
- Use **exact** tool names, variable names, and Power Fx identifiers in instructions.
- **Avoid naming specific knowledge sources directly in instructions — describe them generically instead** (prevents incorrect grounding).
- **Curate the toolkit**: a smaller set of high-quality choices beats an exhaustive overlapping set. Remove/disable irrelevant or risky tools so they don't confuse the planner.
- You may add **1–2 example utterances** inside a topic description to hint the planner ("E.g., 'I forgot my password'") — but keep descriptions succinct.

### Topic architecture
- Two kinds of topics: **entry-point topics** (triggered by utterance/NLU) and **reusable bite-size topics** (called via redirect, with input/output variables). A topic can be both.
- **Prefer many bite-size topics over few large ones** — easier to maintain, and trigger mapping is more precise.
- **Create reusable topics** for shared logic/messages (update once, applies everywhere). If you call the same flow from several topics with similar nodes, make it a dedicated topic.
- **Avoid topic overlap** — frequent "did you mean" (Multiple Topics Matched) calls reveal it. Compare trigger phrases across topics, remove ambiguous pairs, avoid reusing the same words.
- **Create a disambiguation topic** for genuinely ambiguous entries (e.g., a generic `Card` topic that slot-fills `CardType` then routes).
- **Use entities to collapse topics**: one `Order` topic + a `FoodType` entity beats `OrderPizza`/`OrderBurger`/`OrderDrinks`.

### Trigger phrases (classic mode)
- **5–10 per topic**, iterate as you learn; **balance counts across topics** so NLU doesn't overweight one.
- **Short (<10 words)**, complete phrases; **vary sentence structure and key terms**.
- **Avoid single-word phrases** (creates confusion) and don't enumerate every entity variation (NLU generalizes automatically).
- Use **unique verbs/nouns**; start from **real production utterances** rather than invented ones; don't omit stop words deliberately.
- **Entity normalization**: the system normalizes detected entities to their type, so "Create a meeting with Susan" can match a "…with Alex" trigger phrase. Great for recall; be aware it broadens matching.

### Slot filling & entities
- Built-in entities handle many surface forms ("$100", "a hundred dollars"). Use entities **wherever possible** — they make the agent seem smarter and cut questions.
- **Synonyms** expand matching and boost trigger weight; **Smart Matching** adds fuzzy/semantic matching (misspellings, "softball"→"baseball").
- **Regex trick** for same-type collisions: "2 towels and 1 pillow to room 101" confuses the `Number` entity → define `TowelQuantity: [1-9] towel`, `PillowQuantity: [1-9] pillow`, `RoomNumber: [0-9]{3}`.
- For **large/evolving datasets** (products, customers), don't use closed lists — query an external source (Dataverse search supports fuzzy matching + confidence scores) and validate before proceeding.

### Topic inputs/outputs in generative mode
- Define **clear input parameters with descriptive, human-friendly names** ("start date", "email address") — the orchestrator auto-generates the question from them. If the generated question is awkward, refine the input's name/description.
- Add **accepted-value lists or Power Fx validation** to enforce valid data.
- **Return outputs instead of sending messages** where the answer is part of a bigger response — let the orchestrator compose the final message.
- **Avoid double-handling**: don't also feed those outputs back into the LLM as open-ended context ("The result says {summary}") — causes over-generation/repetition.

### Knowledge / RAG
The pipeline: **query rewriting** (clarifies meaning, adds last-10-turn context, makes it search-friendly) → **retrieval** (top **3 results per source**) → **summarization with citations** → **safety/grounding validation**.

- **RAG is for**: answering from knowledge bases, summarizing policies/FAQs/procedures, retrieving specific facts.
- **RAG is NOT for**: full document comparison, policy-compliance evaluation, complex reasoning over long unstructured documents.
- Keep sources **tight and well-described**; document hygiene matters (clean text, headings, split large files).
- See [../reference/knowledge.md](../reference/knowledge.md) and [../reference/dataverse-knowledge.md](../reference/dataverse-knowledge.md).

---

## Step 6 — Autonomous / agentic agents

Autonomy = the agent perceives events, decides, and acts **without a user prompt**, using triggers + instructions + guardrails.

**Implementation best practices**
- **Define clear scope and goals** — state what it accomplishes and **where its authority ends**; a narrow scope prevents "wandering off."
- **Provide quality data and instructions** — garbage in, garbage out.
- **Test thoroughly, roll out gradually** — sandbox/simulate first, then stage; monitor decisions closely early.
- **Human oversight for critical actions** — require approval/confirmation for sensitive operations.
- **Iterate** — expand responsibility in small increments as reliability is proven.

**Security guardrails**
- **Least-privileged access** — read-only if it only needs to read.
- **Input validation & authenticity** — verify trigger events are genuine (sender validation, expected keywords) so attackers can't spoof a trigger; put the agent behind authentication.
- **Robust guardrails/fail-safes** — e.g., "only send an email after checking a knowledge source"; constrain tool parameters (limit recipients to a domain via Power Fx entity validation).
- **Audit logging & monitoring** — log triggers, decisions, actions; alert on anomalous access.

> ⚠️ **Agents with event triggers currently use the MAKER's credentials only** — and tools they call do too. Plan permissions accordingly.

**Lifecycle hooks (custom triggers) for generative agents**
| Trigger | Fires | Use for |
|---------|-------|---------|
| **On Knowledge Requested** | Just before a knowledge query | Intercept/augment retrieval — read the `SearchPhrase`, inject custom results, route to a proprietary index. **Not visible in the UI — enable via YAML by naming a topic exactly `OnKnowledgeRequested`.** |
| **AI Response Generated** | After the draft answer, before sending | Post-process text/citations, swap URLs, append a survey, redact; `ContinueResponse=false` suppresses the original |
| **On Plan Complete** | After the plan finishes and the response is sent | Cleanup, end-of-chat topic, survey — **gate it** so it doesn't fire after every turn |

---

## Step 7 — Agent flows: best practice

- Flows are **deterministic** (same input → same output) and consume Copilot Studio capacity per action.
- Only flows with the **"When an agent calls a flow"** trigger can be added as agent tools.
- **Latency warning:** cloud flows introduce latency. Prefer **direct connector calls** or the **Send HTTP request** node for simple calls. Flows/agent flows are **not recommended as the primary integration for latency-sensitive, high-concurrency IVR/voice** — their synchronous model and throttling add latency under load.
- **Turn on Express mode** (preview) for speed:
  - **Requires**: "When an agent/app calls a flow" trigger, a **Respond to agent/app** action, a Copilot Studio plan, and an environment on the new Power Automate infrastructure.
  - **Use for**: logic-heavy, time-sensitive flows with a response action.
  - **Don't use for**: data-heavy flows (large row sets) or fire-and-forget flows.
  - **Limits**: 2-minute execution, ~**100 actions** (loops count per iteration; Apply-to-each ≤100 items, Do-until ≤100 iterations), **1,024-char variables**, **64 KB** per connector response, **no Delay or webhook actions**, no auto-test in designer, no live activity view.
- Watch **agent flow enforcement**: when prepaid capacity is exhausted, new flow runs are blocked (agent keeps working for non-flow turns). See [../reference/licensing-and-credits.md](../reference/licensing-and-credits.md).

---

## Step 8 — Test, tune, improve (the loop that compounds)

1. **Activity map** during testing — inspect the plan: which topics/tools were invoked, in what order, was the follow-up appropriate? Wrong pick ⇒ refine **names/descriptions** first.
2. **Review transcripts** after publishing — hunt hallucinations and inaccuracies; fix by adding facts to knowledge, tightening instructions, or adding a topic for a gap.
3. **Iterate with small changes, one at a time** — verbose output ⇒ tweak format instructions; over-invoked tool ⇒ narrow its description.
4. **Convert real conversations into test cases** — build regression test sets; use built-in **Evaluate** methods and the **Copilot Agent Kit** for batch testing + rubrics, gated in **pipelines**.
5. **Monitor metrics** — success/resolution rate, **fallback rate**, escalation, CSAT, per-topic/tool usage. A noisy small-talk topic firing constantly? Disable or narrow it.

See [../reference/analytics-evaluation.md](../reference/analytics-evaluation.md).

---

## Step 9 — Plan for throughput & scale (enterprise)

**Rate provisioning ≠ license provisioning.** Licenses/credits cover *entitlement*; rate limits cover *how fast* traffic can be processed before throttling.

- **Use peak windows, not monthly totals.** A campaign hour can throttle an agent whose monthly average looks safe. Build **both average and peak** traffic profiles.
- **Limits apply at many scopes** — environment, tool, API, connector, channel, downstream service. Copilot Studio message limits are **per Dataverse environment** (include every source: channels, integrations, autonomous workloads, skills).
- **The lowest limit in the runtime path determines the experience** — your agent can be fine while a flow, connector, Dataverse call, CLU/AI service, or external API throttles.
- **Treat B2C and autonomous agents as first-class rate-provisioning scenarios** — they burst.
- Throttling also happens from: predictable employee peaks, campaigns/outages/launches, flows with loops/retries/pagination/child flows, synchronous telemetry/transcript export in the turn path, shared environment capacity, and aggressive load tests.

**Design to reduce pressure (before requesting more capacity):**
- Place API/connector calls **strategically**; don't make users wait on multiple completions.
- **Cache retrieved data in variables** instead of repeat calls.
- Prefer **direct connector / HTTP node** over cloud flows; enable **express mode**.
- **Split agents across environments** to avoid competing for environment-scoped limits.
- For autonomous agents: **queues, batching, trigger filters, scheduled processing, retry controls**.
- Move **reporting/audit/telemetry work out of the interactive path**.

If peak estimates still exceed limits, **open a Power Platform support request before UAT/load test/launch** with: environment ID, agent ID, business impact, agent design snapshot, average + peak traffic estimates, session profile, runtime path, and the mitigations already applied. Increases aren't guaranteed.

---

## Golden rules (the short version)

1. **Prototype fast, design before scaling.**
2. **Names > descriptions > instructions** for routing. Curate, don't accumulate.
3. **Decide your control layers** — never let AI own irreversible actions.
4. **Deterministic where it matters, generative where it helps.**
5. **Bite-size, reusable topics**; use entities to collapse variants.
6. **Return outputs, not messages**, so the orchestrator can compose.
7. **Keep knowledge tight and well-described**; RAG is for facts, not deep document analysis.
8. **Autonomy = narrow scope + least privilege + validated triggers + audit + human-in-the-loop.**
9. **Flows add latency** — direct calls/express mode for speed; never the primary path for voice.
10. **Test with the activity map, turn real conversations into regression tests.**
11. **Plan peaks, not averages** — the lowest limit in the path wins.

## Anti-patterns to avoid
- A topic per phrasing variation (topic sprawl) instead of intent + entities.
- Vague/overlapping names and descriptions (the planner picks wrong or picks several).
- Instructions that reference tools/knowledge the agent doesn't have.
- Instructions that alter citations (grounded answers get discarded).
- Letting the planner call irreversible actions with no confirmation.
- Cloud flows in latency-sensitive voice paths.
- Exposing every possible tool "just in case."
- Load testing without rate provisioning.
