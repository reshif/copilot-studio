# Guidance — Plan, Scope & Prove Value

Everything that happens **before and around** the build: defining objectives, choosing architecture, applying responsible AI, and proving ROI. From Microsoft's Plan guidance.

---

## 1. Define objectives and success criteria

Articulate clearly, up front:
- **Business challenges** — what problems does the agent solve?
- **Purpose and features** — main capabilities and expected outcomes.
- **End-users** — who interacts with it, through which channels?
- **Migration** — new solution, or replacing an existing one?

Document **stakeholders**: business sponsor, product owner, technical architect, delivery partner. This creates accountability and clarifies who inputs at each stage.

> **Example scope statement:** *"An Employee Agent is available to all staff in Microsoft Teams and the company portal, answering HR, IT, Sales, and Finance questions using enterprise data sources. It also automates common tasks like account unlocks and meeting room bookings."*

### Target KPIs (make success measurable)
- **Deflection rate** — % of conversations resolved without human intervention.
- **User satisfaction** — target CSAT.
- **Adoption** — users / interactions per month.
- **Cost savings** — reduction in support cost or manual effort.
- **Operational agility** — can SMEs update and deploy autonomously?

> **Example targets:** 60% deflection (from 20%) · CSAT 4.5+ (from 3.0) · 20,000 conversations/month · $1M annual support savings · same-day updates.

Also plan: **iterative, user-focused delivery**, **risk prioritization with workarounds**, a **cross-functional team**, and **throughput/rate provisioning** (see the playbook, Step 9).

---

## 2. Prove business value (ROI)

Microsoft's stance: *"This challenge isn't about better models or bigger deployments. It's about measurement discipline."* **Define value before you build, capture telemetry from day one, and review results regularly with a named sponsor.**

Three questions every business review asks:
1. **Are your agents being used?**
2. **Are they working well for the people they serve?**
3. **Are they returning enough value to justify scaling?**

The signals are already in your tenant — every agent emits usage, quality, and outcome signals from the first conversation.

| Stage | What you produce |
|-------|------------------|
| **Define value before you build** | Four discovery questions, a prioritization grid, a **baseline protocol**, a licensing/consumption plan, and a prebuild governance checklist |
| **Measure impact** | A **four-pillar value framework**, quantitative + qualitative metrics per pillar, **leading and lagging indicators**, Microsoft's published **Agent Assisted Hours formula**, and the adoption levers that move the numbers |
| **Tell the value story** | The **three measurement tiers**, the shape of the **value curve over time**, five steps to adopt this quarter, and an **executive scorecard template** |

Supporting references: **16 use-case blueprints** (each with a worked Agent Assisted Hours example) and an **agent metrics reference library** (definition, use case, and where to read each metric).

> **The economics that justify deflection:** a human-representative contact typically costs **$5–$10**; an agent session that resolves the request costs about **$0.50**. Higher deflection ⇒ direct cost savings *and* better CSAT (shorter waits; reps focus on complex work).

**Baseline before you launch** — you can't show improvement without a "before."

---

## 3. Define your solution architecture

The agent lifecycle is a **closed loop**:
- **Build & Publish** — get to production fast with the right architecture, governance, and channels.
- **Analyze & Improve** — stay valuable by learning from usage and closing quality gaps.

### The 8 components of a real conversational AI experience
1. **Language understanding & orchestration** — intent + entity extraction.
2. **Dialog management** — guided vs open-ended; transitions, interruptions, clarifications, multi-step context.
3. **AI to generate answers or take actions** — search knowledge, summarize, trigger APIs/connectors.
4. **Integration with other systems** — end-to-end processes span systems.
5. **Deployment & runtime** — channels, multi-language, multi-channel publishing.
6. **Security & authentication** — who can invoke it; must the user sign in?
7. **Analytics & continuous improvement** — telemetry, KPIs, transcripts.
8. **ALM** — environments, versions, automated deployment, compliance.

### Architecture layers to document
Runtime (channels, agent-as-skill) · Integrations (HTTP, connectors, workflows, prompts, skills) · ALM (solutions, CI/CD) · Dialog management (topics, actions, inputs/outputs) · Language understanding (classic NLU, built-in, BYO, generative orchestration) · Generative answers (query rewriting, knowledge search, summarization) · Security (secrets, identity, authn/authz, endpoint security, data policies, audit) · Triggers (autonomous, system) · Analytics (standard, telemetry, transcripts).

### Identify technical challenges early
Common roadblocks: secure connections to **on-premises** resources, deploying to channels like **WhatsApp/Slack**, **transcript download**, **multilingual** support — and **throughput/rate limits across the whole runtime path**. Document and validate with stakeholders.

### Two frameworks to align to
- **Architecting agent solutions** pillars: *fit for purpose*, *operability*, *trust/traceability/transparency*.
- **Power Platform Well-Architected** pillars: *Reliability, Security, Operational Excellence, Performance Efficiency, Experience Optimization* — a reference point for validating trade-offs, not just "getting an agent live."

---

## 4. Apply responsible AI

An AI system includes the technology, the people who use it, the people affected, and the environment it's deployed in.

### Four core principles
- **Fairness** — diverse, representative data; minimize bias; audit regularly.
- **Accountability** — clear roles/responsibilities; documented ethical standards.
- **Transparency** — users must **know they're interacting with generative AI**; communicate why AI was chosen, how it's designed, monitored, updated.
- **Ethics** — inclusive team, diverse community input early, regular assessment, governance framework with audits.

> Copilot Studio ships a default disclosure: *"Just so you are aware, I sometimes use AI to answer your questions."*

### Data privacy & security
Understand **platform features/native controls**; data is **encrypted at rest and in transit** (TLS; Microsoft backbone between Dynamics 365/Power Platform/Azure OpenAI); apply **RBAC via Entra ID** and **least privilege**; **monitor and audit** access; ensure **GDPR/HIPAA/CCPA** compliance; **train users** on privacy practices.

> Data is provided to an agent **based on the access level of the current user** — security trimming is a feature, not an afterthought.

### Bias awareness & mitigation
Diverse/representative data + regular audits · bias-detection tools and fairness metrics · debiasing (resampling, reweighting, adversarial) · **human-in-the-loop** review and an ethics/governance board · transparency · continuous monitoring and retraining.

### Ongoing monitoring
Establish **feedback loops** so users can report inaccuracies, and maintain **audit logs** of data access/modification.

---

## Related
[agent-design-playbook.md](agent-design-playbook.md) · [govern-operate-and-improve.md](govern-operate-and-improve.md) · [../reference/licensing-and-credits.md](../reference/licensing-and-credits.md) · [../reference/analytics-evaluation.md](../reference/analytics-evaluation.md)
