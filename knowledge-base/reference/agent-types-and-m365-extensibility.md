# Reference — Agent Types, Templates & Microsoft 365 Copilot Extensibility

Copilot Studio isn't the only way to build a Microsoft agent, and it produces more than one *kind* of agent. Knowing the landscape prevents building the wrong thing.

## The agent-type landscape
| Type | Built with | Nature | Where it runs |
|------|-----------|--------|---------------|
| **Custom agent** (Copilot Studio) | Copilot Studio (full authoring) | Full: instructions + knowledge + tools + topics + triggers + child/connected agents | Any channel (Teams, web, M365 Copilot, WhatsApp…) |
| **Agent for Microsoft 365 Copilot** (= **declarative agent**) | Copilot Studio (M365 Copilot page) **or** Agent Builder | A prompt + knowledge + tools over the **M365 Copilot** model/orchestrator | Inside M365 Copilot / Teams |
| **Custom engine agent** | Copilot Studio / Agents SDK | Brings its own model/orchestration | M365 Copilot + other channels |
| **Agents SDK agent** (pro-code) | Microsoft 365 Agents SDK / Agents Toolkit (VS Code) | Fully custom code | Any; connectable to Copilot Studio |

**Declarative agent** = "Agent for Microsoft 365 Copilot": you author a **prompt** defining behavior/personality/rules; it rides on M365 Copilot's model, knowledge, and orchestration. **Custom engine agent** = you supply the model/orchestration (Copilot Studio custom agents, or Agents SDK).

### Choosing
- **Need M365 Copilot's model + your grounding/tools, minimal build** → *Agent for M365 Copilot* (declarative) / Agent Builder.
- **Need full control** — topics, autonomous triggers, multi-channel, deterministic flows, own orchestration → *Copilot Studio custom agent* (this KB's focus).
- **Pro-code / custom runtime, connect into Copilot Studio** → *Agents SDK* agent (connect as an external agent — see [../09-multi-agent-orchestration.md](../09-multi-agent-orchestration.md)).

## Agents for Microsoft 365 Copilot (details)
Created from the **Microsoft 365 Copilot** page (not the main Agents list; won't appear there). Config: name (≤42 chars, no `<>`), PNG icon <72 KB, description, **instructions**, optional **suggested prompts** and **knowledge**, **Web browsing** toggle (Bing).

- **Knowledge**: SharePoint (path to site/library root — not a single file), **Copilot (Graph) connectors**, Web browsing. Uses the **user's** credentials (sees only what they can access).
- **Tools**: Prompt, Agent flow, Computer use, Custom connector, MCP, REST API — same guided experience as custom agents (Details/Inputs/Completion). First trigger shows a **connection card** for sign-in (own creds or SSO).
- **Suggested prompts** — starter prompts; also a place to expose sophisticated capabilities users wouldn't discover.
- **Publishing**: **not auto-deployed** — publishing populates the org catalog entry (Office/Teams catalogs + Integrated Apps). A **bot resource** is provisioned in Entra ID. **Availability options**: Share Link (deep link), Show to teammates/security groups, Show to everyone (admin submits to org catalog), Download as `.zip`.
- **Caveats**: M365 Copilot **caches** answers per session (use **Start new test session** to bust). `-developer on` in chat explains tool selection. Runtime results can differ from the test panel/Teams. **These agents don't feed the Copilot Studio Analytics page.** Prereq: makers/users need **M365 Copilot licenses**.
- **Security**: tools pull untrusted data (emails/tickets) → prompt-injection risk; use DLP-secured connectors (see [governance-security-dlp.md](governance-security-dlp.md)).

## Publishing a Copilot Studio **custom** agent to M365 Copilot / Teams
Add the **Teams + Microsoft 365 channel** (see [publishing-security-limits.md](publishing-security-limits.md)). Custom agents get full channel reach and **do** collect analytics.

## Prebuilt agents, templates & the Agent Library
Don't always build from scratch:
- **Agent templates (bundled)** — Agents page → **Start with an agent template** → pick one. Templates come with preconfigured **instructions, tools, topics, triggers** to customize. (See *Create a custom agent from a template*.)
- **Prebuilt agents** — ready-made agents Microsoft ships for common scenarios.
- **Agent Library (Copilot Agent Kit)** — from **April 2026 release**; a centralized hub to **discover, install, manage** prebuilt **templates and reusable components** (tabs: All / Templates / Components). Install into your environment and customize.
- FAQ: *agent templates and managed agents*.

### Guidance for our framework
- **Start from a template** when one matches the domain — faster, and you can clone it to VS Code and study the generated YAML.
- Treat **reusable components** (Agent Library) as building blocks; keep our own component conventions ([../11-code-first-framework-and-workflow.md](../11-code-first-framework-and-workflow.md)) aligned so we can mix them.
- Decide agent *type* first (custom vs declarative vs SDK) — it determines channel reach, analytics, and how much we author.
