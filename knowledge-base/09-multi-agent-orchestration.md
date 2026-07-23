# 09 — Multi-Agent Orchestration

Copilot Studio agents can be extended by connecting them to **other agents**. Agents can hand off interactions or respond to autonomous triggers, letting you scale with **modular agents** tailored to specific tasks/data. All added agents appear on the main agent's **Agents** page.

## Ways to add other agents

- **Child agents** — lightweight agents *inside* your existing agent.
- **Connected Copilot Studio agents** — connect to other Copilot Studio agents **in the same environment**.
- **External agents**:
  - **A2A protocol** agents (agent-to-agent).
  - **Microsoft Foundry** agents *(preview)*.
  - **Fabric Data** agents *(preview)*.
  - **Microsoft 365 Agents SDK** agents *(preview)* — pro-code agents built with the SDK.

> Connecting to Foundry, Fabric, and M365 Agents SDK agents is currently **public preview**.

## Child agents vs connected agents — when to use which

**Use CHILD agents when:**
- Single use case / single intent or task (create a ticket, check status, book a flight).
- One developer or a small cohesive team owns the whole solution.
- You want to logically group tools/instructions/knowledge into subagents within one larger agent.
- You DON'T need separate settings, auth, or deployment per subagent.
- You DON'T intend to publish them separately or reuse across agents.

**Break into CONNECTED agents when:**
- The main agent can no longer reliably differentiate its tools by name/description — rule of thumb: **more than ~30–40 choices** (tools + topics + agents), or fewer but with **similar descriptions**.
- Multiple teams manage different agents independently.
- Agents need **separate publishing**, their own **channels**, and independent **ALM**.
- Agents need their own **settings** (e.g., different model).
- You want the agent to be **reusable** by more than one main agent.

You can **mix** child and connected agents. E.g., break out reusable pieces as separate agents (also directly accessible), each with their own child agents.

## Trade-offs / impacts

- **Latency**: extra orchestration hops. Main agent's orchestration picks a connected agent → that agent runs **its own** orchestration to handle the query.
- **Bigger surface**: more testing, management, and governance.
- Start by improving **descriptions/differentiation** before splitting.

## How agents are invoked

1. **Generative orchestration** — the model routes to a child/connected agent based on its name/description (like a tool).
2. **Explicit redirect from a topic** — add an agent-redirect node; when the agent finishes, the originating topic resumes; you can add more nodes after. Some agents support passing **inputs** and reading **outputs** (each output auto-creates a topic variable).
3. **Reference in instructions** — type `/` in the **Instructions** field and pick an agent, to break long instructions into focused pieces (useful for autonomous agents). The referenced agent shows in the test panel's activity map.

## Managing connected/child agents

- **Enable/disable**: the **Enabled** toggle on the Agents page makes an agent momentarily inactive (won't respond to users/triggers).
- **Delete child agent**: `…` → **Delete**.
- **Remove connected agent**: `…` → **Disconnect agent**.

## Known limitations (important)

- **Fabric Data agents**: can't be redirected to via a topic **Redirect** node; can't be referenced in instructions; don't function when the main agent is deployed to **Microsoft 365 Copilot**.
- **Citations** may not always be preserved when passing outputs back to the calling agent.
- Child/connected agents **respect the main agent's "Use general knowledge"** setting.
- **Reuse rule**: an agent that is itself a *main agent with connected agents* **can't also be used as a connected agent** elsewhere. But any agent **without** connected agents can be a connected agent in multiple main agents.
- In parent-relies-entirely-on-subagents setups with "Don't respond" after-child behavior, the runtime may emit a system `explanation_of_tool_call` message — expected, not a misconfiguration.

## In the cloned YAML

- Child agents appear under `agents/<Name>/agent.mcs.yml` with `kind: AgentDialog`.
- Connected-agent actions surface under `actions/` (`kind: TaskDialog`).
- The code-first `add-other-agents` skill notes **child agents must be pushed/pre-pushed** before you can add knowledge to them.

## Design heuristics for our framework

- **Default to a single agent** with well-described tools/topics; only go multi-agent when routing degrades or org/ALM boundaries demand it.
- Keep each agent's **tool/topic/agent count under ~30–40**; enforce distinct descriptions.
- Treat reusable capabilities (e.g., "IT ticketing") as **standalone connected agents** with their own ALM.

Related: Microsoft's **multi-agent orchestration patterns and best practices** guidance. See [12-glossary-and-sources.md](12-glossary-and-sources.md).
