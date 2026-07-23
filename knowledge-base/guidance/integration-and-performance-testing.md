# Guidance — Integration Strategy & Performance Testing

How agents reach other systems, and how to prove they hold up under real traffic.

---

## Part 1 — Choosing an integration pattern

Patterns **aren't exclusive — combine them**. And remember: *integrations are only as fast as the endpoints they connect to.*

### Power Platform connectors
Actions (read/write/update) and triggers exposed through a low-code interface. **Prebuilt** (large SaaS ecosystem) or **custom** (no/low-code wrapper for REST APIs).

**Good for:** maker familiarity · Power Fx support (variables, conditions, parameters) · **built-in parsing and error handling** · zero dev time for prebuilt · custom connectors are **build-once, reuse-everywhere** · App Insights monitoring · some support virtual networks.

**Watch out:** returning **hundreds of results causes significant response delay** · third-party connectors prompt users for credentials — consider **maker credentials** when users won't have them.

### HTTP requests
Specify URI, method, headers, body each time.
**Good for:** services with no prebuilt connector · **faster to build than a custom connector**.
**Watch out:** many low-code makers can't configure or support HTTP nodes · **not shareable/reusable** across the org like a custom connector.

### Agent flows
A predefined sequence of actions using connector actions; agents pass inputs and receive outputs.

**vs Power Automate cloud flows:** agent flows enable **high throughput and low latency**, offer extra capabilities like **human-in-the-loop** actions, and consume **Copilot credits** (cloud flows use per-user/per-process licensing).

**Good for:** no/low-code · **deterministic** chaining · separately monitorable · **hides secrets/keys** and pulls credentials from Azure Key Vault · handles **large data volumes and file sizes** · **concurrency/parallel execution**.

**Watch out:** performance bounded by the APIs it calls · **you must design an error-handling pattern** so the agent knows how to handle exceptions · the **response to the agent has a size limit** · longer-running logic can keep executing *after* the Respond-To-Agent action.

### Bot Framework skills
Pro-code, reusable conversational building blocks; register then call from a topic node.
**Good for:** **synchronous execution** · **private endpoint support** · reusing existing skills.
**Watch out:** pro-code ongoing cost (e.g. C#) · runs in Azure AI Bot Service (**extra Azure cost**) · **ALM lives outside Power Platform**.

### When nothing is fast enough
If agent flows / Copilot Studio can't execute logic quickly enough, consider **Dataverse custom APIs**, **Dataverse low-code plugins**, or **Azure Functions**. Some scenarios need a **middle layer** that transforms messages as it relays them.

### Three ways to attach an integration
- **Tools** — describe in natural language *when* the orchestrator should pick it; define input derivation (AI-filled or explicit/Power Fx) and output shape; invoked as part of a generated plan.
- **Topics** — called at a fixed position in the sequence every time; inputs via Power Fx.
- **Agents** — a child agent with its own instructions/knowledge/tools, or a connected agent (Copilot Studio, M365 Agents SDK, Foundry, Fabric, A2A).

### Integration planning table (do this per system)
Capture: **connected system · called by · called with · expected daily volume · expected peak · API calls per minute · details**.

| Connected system | Called by | Called with | Daily | Peak | API/min | Details |
|---|---|---|---|---|---|---|
| ServiceNow | Service Desk KB topic | Workflow | 5,000 | 1,000 | 600 | Query returns JSON for generative-answers custom data |
| Contoso website | Knowledge | Generative answers | 1,000 | 100 | — | `/en-us/` set dynamically from user locale |
| Internal Directory API | Conversation Start topic | Workflow | 15,000 | 5,000 | — | Custom connector inside an Azure VNet |
| Weather API | Weather topic | HTTP | 500 | 100 | — | Simple GET |
| Teams | Meeting Booking topic | Tools | 1,500 | 300 | 100 | Create meeting from conversation inputs |
| SAP | Every 24 hours | Workflow | 20,000 | — | 2,500 | Nightly catalog sync to Dataverse |

> Also determine limits you can't look up: **OpenAI capacity rate limits aren't documented**, and Bot Framework skills depend on the Azure services you chose.

---

## Part 2 — Performance & load testing

The platform auto-scales, **but your custom logic and backend APIs may not**. An agent can behave perfectly in development and fail under real traffic.

### ⚠️ Test safely
- **Mimic realistic user behavior** — generated load beyond expected behavior can cause **message-consumption overage and environment throttling**.
- Ensure the tenant/environments have **sufficient licenses and billing policies**.
- **Confirm throughput headroom first.** If estimates suggest default limits could be exceeded, **open a support request before testing begins** (see the playbook's throughput section).
- Design for **baseline** and **anticipated heavy load** — **don't design deliberate stress tests** that push the service to non-responsive.

### Step 1 — Model user behavior per use case
Capture what users say, how many drive each use case, and the **engagement pattern** (steady vs all-at-once).

| Use case | Common utterances | Engagement pattern |
|----------|-------------------|--------------------|
| Loan application | "I need a new loan", "apply for a loan" | ~1,000 concurrent, spread through the day |
| Balance inquiry | "What's my account balance?" | 10,000 concurrent, **all around midday** |

### Step 2 — Write the test plan
Minimum contents: **objective · scope · KPIs · test scenarios · test data · tools · success criteria**. Reuse conversational scenarios from your evaluation test cases or the Copilot Agent Kit.

**Example plan (banking agent)**
- **Objective:** evaluate performance under baseline and load.
- **Scope:** in — baseline + load testing; out — stress testing.
- **KPIs:** response time; error rate.
- **Baseline:** Loan application, 1,000 concurrent users, 15 min.
- **Load:** Loan application 1,000 users/15 min; Balance inquiry 10,000 users/5 min.
- **Tools:** Apache JMeter + its built-in reports.
- **Success criteria:** *Baseline* — 95% of responses **< 2s**, error rate **< 0.5%**. *Load* — 90% **< 3s**, error rate **< 1%**.

### Step 3 — Simulate MULTI-TURN conversations (essential)
Single-message tests hide the real problems. In a realistic flow, the first message only touches intent recognition (**sub-second**), while a later message triggers a **backend API call** that adds real latency — and some long-running actions only fire after a specific sequence of choices.

So: include **series of utterances that drive complete conversational flows** in your test data, and make sure scripts send **multiple utterances within a single conversation**.

---

## Part 3 — Custom analytics strategy (when built-in isn't enough)

Built-in reports cover performance/usage, CSAT, session info, topic usage, and billed sessions. Go custom when you need to **share with stakeholders**, **report beyond the default 30-day transcript retention**, or **build something not covered out of the box**.

- Analytics come from an internal Copilot Studio service; usage is **also written to Dataverse** (`conversationtranscript`).
- **Default retention: 30 days** (changeable for transcripts in Dataverse).
- Tables: `bot` (small), `botcomponent` (small), `conversationtranscript` (**large**, scales with usage).
- **Recommended long-term pattern:** export to **Azure Data Lake Storage Gen2 via Azure Synapse Link for Dataverse** (incremental sync, Common Data Model). Select `ConversationTranscript` — the other two tables don't support incremental sync.
- ⚠️ **Synapse Link mirrors deletions** — the bulk-delete job that purges 30-day-old transcripts also removes them from the lake. Use **snapshots** or **append-only mode**.
- Also consider **AI-powered custom metrics** (natural-language defined, shown as donuts) and the **Copilot Agent Kit** (Agent Inventory, Conversation Analyzer, Conversation KPIs with long-term retention and extensible Power BI).

## Related
[agent-design-playbook.md](agent-design-playbook.md) (throughput planning) · [../reference/metrics-and-roi.md](../reference/metrics-and-roi.md) · [../reference/tools.md](../reference/tools.md) · [../reference/prompts-and-agent-flows.md](../reference/prompts-and-agent-flows.md)
