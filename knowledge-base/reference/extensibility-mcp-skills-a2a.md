# Reference — Extensibility: MCP, Skills & A2A

Three ways to plug external capability/agents into a Copilot Studio agent, beyond connectors/flows/REST.

## Choosing the integration model
| Need | Use |
|------|-----|
| Call an API / basic HTTP service | Custom connector / HTTP tool ([tools.md](tools.md)) |
| Reusable tool set (functions + data) from a server | **MCP server** |
| Reuse a pro-code bot as multi-turn conversational capability | **Skill** (Bot Framework / Agents SDK) |
| Delegate a **task** to an autonomous external agent | **A2A** connected agent |
| Integrate a Microsoft 365 Agents SDK agent | Activity Protocol (connected agent) |

You can combine several in one agent.

---

## Model Context Protocol (MCP)
Connect to an **MCP server** to expose its **Tools** (callable functions) and **Resources** (file-like data the agent can read); MCP also defines Prompts (Copilot Studio currently supports **tools + resources**).

- **Requires generative orchestration ON.**
- Each tool/resource the server publishes is **automatically available** (name, description, inputs, outputs provided by the server). Changes on the server (add/remove) are **reflected dynamically** — always latest, obsolete ones removed. One server can host many tools.
- Add via: **MCP onboarding wizard** → connect existing server, or **create a new MCP server**, then **add its tools/resources** to the agent. Config page shows **Tools** and **Resources** sections (instead of Inputs/Completion).
- **Publish an MCP connector** (custom-connector certification) to share across tenants.
- **DLP:** blocking the underlying Power Platform connectors **also blocks MCP tools** that rely on them (see [governance-security-dlp.md](governance-security-dlp.md)).
- You're responsible for tools/resources accessed from non-Microsoft servers.

---

## Bot Framework / Agents SDK Skills
A **skill** = an existing pro-code bot (Microsoft 365 Agents SDK, or legacy Bot Framework SDK) registered into a Copilot Studio agent and invoked from **topics**. Good for **complex, multi-turn** server- **and** client-side actions (e.g. schedule a meeting, navigate a page).

- **Skills vs agent flows:** flows = simple single-turn maker-built automation with an agent response; skills = complex multi-turn, any supported response (adaptive cards, client events), built/hosted by **developers**.
- **Requirements:** Agents SDK v1.0.0+ (or legacy Bot Framework SDK 4.12.0+, but new skills should use the Agents SDK). App registration must be **single-tenant**. Teams-app authoring needs a **standalone subscription**.
- **Register:** Settings → **Skills** → **Add a skill** → copy your **agent ID** (give it to the skill dev for the skill's **allow list**) → enter the **skill manifest URL** → validate → use in topics.
- **Manifest** must be valid JSON ≤ **500 KB**, with `name`, `msaAppId`, a single `endpoint`, and `activities` (`id`, `description`, `type` = only `event` or `message`). Endpoint origin must match the Entra app's **Publisher domain** / Home page URL.
- **Limits:** 100 skills/agent; 100 actions/skill; 25 inputs & 25 outputs per action.
- **Compliance:** skill must be registered as an app in the signed-in user's Entra tenant.
- **DLP:** block via the **"Skills with Copilot Studio"** connector.
- Common validation errors: `MANIFEST_FETCH_FAILED`, `MANIFEST_MALFORMED`, `MANIFEST_ENDPOINT_ORIGIN_MISMATCH`, `APPID_NOT_IN_TENANT`, `ENDPOINT_HEALTHCHECK_UNAUTHORIZED` (agent not on the skill's allow list).

---

## Agent2Agent (A2A) protocol
An **open standard** for agent-to-agent communication. Copilot Studio can **delegate tasks** (not just call APIs) to any external agent implementing A2A — external frameworks, hosted anywhere, with their own reasoning.

**A2A vs HTTP connector:** A2A is designed for agent workflows, **multi-turn**, rich contextual metadata, and cross-framework interoperability; HTTP is not.

### Connect (steps)
1. Agents page → **Add an agent** → **Connect to an external agent → Agent2Agent**.
2. Enter the **endpoint URL** (the message endpoint, *not* the agent-card URL). If a valid **agent card** exists at `…/.well-known/agent.json`, Copilot Studio auto-fills **name + description**; else enter them manually (description drives when the main agent delegates — write good metadata).
3. **Authentication**: None / **API key** (header or query param name) / **OAuth 2.0** (client ID, secret, authorization/token/refresh URLs).
4. Create → pick/create a **connection** → **Add and configure**.

> A2A uses the **custom-connector infrastructure**, so it can also reach agents on-premises / in a VNet.

### How it works
The orchestrator delegates a natural-language task to the A2A agent, which returns a response. The A2A message payload carries structured **metadata**: a unique `contextId`, message IDs, locale, and the **full chat history** (`copilotstudio.microsoft.com/a2a/chathistory`), plus content parts (text/tool calls).

### Your responsibilities (external agents)
Review/test data flows and handling; ensure quality/security/trust; provision permissions/boundaries/approvals; ensure observability, identity/traceability, and human oversight.

> A2A sits alongside the other multi-agent options (child agents, connected Copilot Studio agents, Foundry, Fabric, M365 Agents SDK). See [../09-multi-agent-orchestration.md](../09-multi-agent-orchestration.md) and [../guidance/multi-agent-patterns.md](../guidance/multi-agent-patterns.md).
