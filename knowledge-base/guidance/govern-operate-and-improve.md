# Guidance — Govern, Operate & Improve

The Manage + Improve half of the lifecycle: zoned governance, ALM discipline, a real testing strategy, and the metrics that tell you whether the agent works.

---

## 1. Zoned governance (segment by risk, not by team)

**Zoned governance** = segmenting environments and applying different policies based on an agent's **purpose and risk level**. Environments are the logical containers that define data boundaries, security roles, data policies, and lifecycle separation.

| | **Zone 1 — Citizen Development** | **Zone 2 — Partnered Development** | **Zone 3 — Professional Development** |
|---|---|---|---|
| **Purpose** | Personal/team productivity agents on content the maker already has; experimentation with safe defaults | Team/department agents built by trained citizen devs **with coach + IT oversight** | Mission-critical, enterprise-grade agents; pro-dev/IT-led; can carry SLAs |
| **Tools** | Agent Builder (M365), SharePoint agents | **Copilot Studio** | Microsoft Foundry Agent Service, **Copilot Studio** |
| **Secure** | Only M365 + Power Platform connectors; agents run **in user context only**; read-only; private | Zone-specific **advanced connector policies**; shared approved data sources; scale via **environment groups + rules** | Zone-specific advanced connector policies + **Microsoft Purview** |
| **Govern** | **Developer environments**; **environment routing** isolates to the maker; **sharing disabled** | Admin-approved environment provisioning; scoped roles/sharing; **ALM pipelines**; **IT approval to publish** | Sharing managed through Integrated Apps in the Microsoft admin center |
| **Monitor** | Copilot hub in PPAC | Microsoft admin center + Purview + PPAC | Microsoft admin center + Purview + PPAC |

### Control levels
**Feature controls**
- **Tenant** — data policies across all environments (unauthenticated usage, channels, knowledge sources, connectors, skills, App Insights); allow/block publishing of **generative-AI agents**.
- **Environment** — scope data policies; allow/block **public data source (Bing)**; allow/block generative AI where no regional Azure OpenAI capacity; **network isolation** (VNet, IP firewall).
- **Agent** — enable/disable generative orchestration, AI knowledge, generative answers, intelligent topic authoring; set **authentication mode**; enforce **web channel security**.

**Maker access controls**
- **Tenant** — licensing gates authoring access; allow/block **self-service trials** and the **Teams app**.
- **Environment** — security-group access; CRUD permissions via security roles (Dataverse tables: *Copilot*, *Copilot Subcomponent*, *Conversation Transcript*).
- **Agent** — share/unshare for co-authoring; System Administrators can read/update **all** agents and transcripts.

### Securing a project (concrete steps)
- **Assign licenses and environment access through Entra ID groups**, not individuals. Only agent authors and just-in-time admins should reach environments and data stores.
- **Assign security roles via Entra group teams** inside each Dataverse environment.
- **Apply restrictive data policies** — block every connector, channel, and setting the project doesn't need.
- **Enable only relevant tenant/environment/agent settings** — store secrets in **Azure Key Vault**; require strong auth on connections; use delegation/impersonation/filtering so data is accessed in the **end-user's** context.
- **Gate releases to production** with reviews.
- Layer on: comprehensive **audit logs**, **CMK**, **Customer Lockbox**, **IP firewall**, **network isolation**, **MFA**, **continuous access evaluation**.

#### Security roles for authoring
| Role | Copilot (`bot`) | Copilot Subcomponent (`botcomponent`) | Conversation Transcript |
|------|-----------------|----------------------------------------|-------------------------|
| **System Administrator** | Organization (CRUD) | Organization (CRUD) | Organization (CRUD) |
| **System Customizer** | Organization (CRUD) | Organization (CRUD) | None |
| **Environment Maker** | User (CRUD) | User (CRUD) | None |
| **Bot Transcript Viewer** | None | None | User (Read) |
| **Bot Contributor** | User (Read) | User (CRUD) | None |

**Environment Maker** is the default role for authors. Creating an agent also creates a **team** and shares the agent + subcomponents with it; transcripts are implicitly shared with that team but **only readable by users with Conversation Transcript read access**.

#### Policy strictness by environment type
| Environment | Data policies | Managed Environment rules | Effect |
|-------------|---------------|---------------------------|--------|
| **Personal development** | Strict | Strict | No unauthenticated agents; most channels/connectors blocked |
| **Dedicated development** | Relaxed | Strict | No unauthenticated agents, but desired channels/connectors allowed |
| **Test & production** | Relaxed | Relaxed | Unauthenticated agents across all channels, unlimited users (post-review) |

#### Sharing rules & residency
**Sharing rules** control whether owners/editors can grant **Editor** (edit, share, publish, use) or **Viewer** (use only) permissions — block it and only admins can. **Set a maximum viewer count** to prevent broad sharing, and you can restrict sharing to individual users (excluding security groups).
**Data residency**: choose where data is stored to meet regional rules; Copilot Studio is designed for **GDPR** compliance with encryption at rest/in transit. Admins can **enable/disable cross-geo data movement** for generative AI features.

#### Mission-critical hardening
**Virtual networks** (connectors reach internal resources without public exposure) · **IP firewall** on Copilot Studio endpoints · **continuous access evaluation** (force token refresh on critical events) · **secrets required** to exchange for a conversation token.

### The governance phases
1. **Capture governance requirements** — align execs, IT, business owners; translate outcomes into requirements.
2. **Implement zoned governance** — layered environments, guardrails, approval workflows separating experimentation from production.
3. **Secure the projects** — RBAC, data policies, gated releases, safe sharing/publishing rules, data residency/compliance.
4. **Design a testing strategy** (below).
5. **Monitor operations, compliance, capacity** — analytics, transcript review, feedback tools; spot regressions.
Plus a **manage checklist** to validate security, governance, compliance, and deployment.

---

## 2. ALM discipline

**Benefits:** reliable releases · governance/compliance · scalability and reuse · quality at speed · business continuity · team collaboration · end-to-end discipline.

**Core practices:** define an **environment strategy**; use **solutions** as transport containers; use **environment variables** for env-specific settings and **connection references** for env-specific connections; implement **CI/CD**; enable **Git integration**.

**Environments:** at least three — **development, test, production**. Production = *production* type; dev/test = **sandbox** type. **Secure every environment with an Entra security group.**

### The ALM golden rules
- Don't customize **outside** a development environment.
- **Always work in the context of solutions.**
- Use a **custom publisher and prefix**.
- Create separate solutions only when components must deploy **independently**.
- Use **environment variables** for settings/secrets that vary by environment.
- Export/deploy solutions as **managed** (except into dev).
- Automate ALM for source control and deployment.

### ⚠️ Copilot Studio items that are NOT solution-aware
These need **post-deployment steps** in the downstream environment:
- Azure **Application Insights** settings
- **Manual authentication** settings
- **Direct Line / Web channel security** settings
- **Deployed channels**
- **Sharing** (with makers or end users)

### Component collections (reuse)
A **component collection** is a reusable set of agent components (topics, knowledge, actions, entities) shareable across agents in an environment — and movable across environments via solutions. Enables multiple teams to own parts of an agent with **independent release cadences**.
*Best practices:* reuse over duplication · version control for consistency and rollback · modular design (e.g., separate IT and HR collections) · independent release cadence · keep collection **versions aligned** across DEV/TEST/PROD.

### Choosing ALM tooling
| | **Azure DevOps** | **GitHub Actions** | **Power Platform Pipelines** |
|---|---|---|---|
| Best for | Enterprise teams needing full ALM control | Dev/admin teams across many environments | Empowering citizen developers |
| Capabilities | Repos + CI/CD, Power Platform Build Tools, Dataverse Git integration | Import/export solutions, deploy downstream, provision environments, Solution Checker, env backup/restore/copy/reset | Centralized deployment visibility, pipeline + security management, quick setup |
| Setup complexity | **High** | **Moderate** | **Low** (minutes) |

---

## 3. Testing strategy

Treat testing as **continuous**, not a gate at the end. Always test before production; automate in the pipeline.

| Test type | What it validates |
|-----------|-------------------|
| **Development-time** | Unit testing of individual components while building |
| **Core scenario** | The "happy path" for core functionality |
| **Knowledge** | Domain knowledge via specific questions |
| **Regression** | Previous cases still pass after changes |
| **Adversarial** | Edge cases, unknown intent, false information |
| **Performance & load** | Latency/scalability under high volume |
| **Security & compliance** | RBAC, conditional access, sensitivity labels, no data leakage |
| **Accessibility & UX** | Clarity, tone, inclusivity, multi-language, accessibility standards |

**Four principles:** **shift-left** (start early) · **iterate and define** (update cases as features/knowledge grow) · **test before production** (validate in staging) · **automate where possible** (CI/CD).

**Tools:** built-in **test sets** (text match, similarity, quality — relevance/groundedness/completeness/abstention); questions can be **generated from the agent's instructions, capabilities, and knowledge**, or **populated from past test-chat conversations**. **Copilot Agent Kit** adds bulk testing (response match, topic match, multi-turn) and rubrics.

**Pre-production security checks:** validate environment data policies/roles/connections; review Azure app registrations, VNets, keys, endpoints; confirm **production** knowledge sources/documents are referenced (not the dev ones).

---

## 4. Measure: engagement, outcomes, deflection

### Sessions
A conversation generates **one or more analytics sessions** (a new session starts when the user has new questions after a path completes). Analytics sessions ≠ **billed** sessions.

- Sessions start **unengaged**.
- A session becomes **engaged** when the user enters a **custom topic** or the **Escalate** topic.
- The **last custom topic** triggered (or first system topic, if none) is associated with the session.
- ⚠️ Proactive greetings / website placement **inflate total and unengaged session counts**.

### Outcomes
| Engagement | Outcome | Definition |
|---|---|---|
| Unengaged | **None** | All unengaged sessions |
| Engaged | **Escalated** (`HandOff`) | Escalate topic triggered or Transfer-conversation node reached |
| Engaged | **Resolved** (`Resolved`) | End of Conversation triggered and user confirms success (`impliedSuccess=FALSE`) or doesn't answer and times out (`impliedSuccess=TRUE`) |
| Engaged | **Abandoned** (`Abandoned`) | Engaged session that ends without resolving or escalating (after ~1 hour) |

**To measure outcomes properly, end conversations with the End of Conversation topic** — it asks the user to confirm resolution and offers escalation, and triggers the **CSAT survey (0–5)**. If you already know the outcome from topic logic, set it directly in YAML:
```yaml
conversationOutcome: ResolvedConfirmed   # or ResolvedImplied
```

### Deflection metrics
| Metric | Definition |
|--------|-----------|
| **Total Sessions** | All analytics sessions in the period |
| **Engagement Rate** | % of total sessions that are engaged |
| **Resolution Rate** | % of engaged sessions resolved |
| **Escalation Rate** | % of engaged sessions escalated |
| **Abandon Rate** | % of engaged sessions abandoned |
| **CSAT** | Average survey score |

**Deflection** = % of requests self-served that a human would otherwise handle. Every org defines it slightly differently (some include Abandon, some only Escalation) — but **raising Resolution and lowering Escalation** moves it directly. Optimization loop: escalation analysis → enrichment → confusion analysis → alternate escalations → transcript analysis.

---

## 5. Improve: evaluation-driven triage

Use this framework once you have pass/fail results across evaluation sets, when:
- An evaluation set scores **below threshold**.
- Specific cases fail and the **root cause is unclear**.
- Scores **improve in one area but regress in another**.
- **Multiple sets fail** and priorities are unclear.
- Behavior **changes unexpectedly after an update**.

It converts scores, failed cases, and regressions into **clear, prioritized actions**. Prerequisite: evaluations are already set up and running.

> Pair with: the **activity map** for per-turn routing, **transcripts** for real failures, and the **Agent Kit** hubs (Agent Debugger, Conversation Analyzer, Insights Hub) — see [../reference/tooling-coe-and-agent-kit.md](../reference/tooling-coe-and-agent-kit.md).

---

## Operating checklist
- [ ] Zones defined; agents placed by risk, not convenience.
- [ ] DLP policies scoped per environment; connector groups reviewed.
- [ ] Dev/Test/Prod exist; prod is *production* type, others *sandbox*; all secured by Entra groups.
- [ ] Everything built inside **solutions**, custom publisher/prefix, deployed **managed**.
- [ ] Environment variables + connection references for anything env-specific.
- [ ] **Non-solution-aware items** (App Insights, manual auth, channel security, channels, sharing) re-applied post-deployment.
- [ ] Test suite covers all 8 test types; automated in the pipeline.
- [ ] End of Conversation topic used so outcomes/CSAT are measurable.
- [ ] Baseline captured; KPIs tracked; triage framework in use.
