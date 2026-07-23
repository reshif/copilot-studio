# Guidance — Agentic AI Adoption Maturity Model

Most AI initiatives succeed as pilots and stall before enterprise scale. This model gives a structured way to answer: *Where are we? What's needed next? What must exist before we increase agent autonomy?*

It looks holistically across **strategy, process transformation, governance, value realization, architecture, operations, organizational readiness, and responsible AI** — not just technology. Based on the **Capability Maturity Model (CMM)** and aligned with Microsoft's **Agent Readiness Framework**.

> Agentic AI adds autonomous decision-making, multistep orchestration, and human–agent collaboration — which requires a **new enterprise operating model**, not just new tools.

---

## Five maturity levels

| Level | Name | What it looks like |
|-------|------|--------------------|
| **100** | **Initial** | Unplanned, experimental. Capabilities inconsistent, siloed, dependent on individuals rather than repeatable practice. |
| **200** | **Repeatable** | Early patterns emerging. Teams can repeat some activities, but approaches are informal and uneven. |
| **300** | **Defined** | Capabilities formally defined, documented, supported by governance, standards, and operating models. Initiatives align to business goals. |
| **400** | **Capable** | Agents embedded in enterprise planning and operations. Process, governance, and technology support scaling and cross-team work. |
| **500** | **Efficient** | Agent-first enterprise. Optimized, continuously improved, backed by leadership, culture, and trust. |

## Five capability pillars

1. **AI strategy and experience** — aligning AI to business objectives, leadership priorities, long-term strategy, and UX goals.
2. **Business strategy** — redesigning end-to-end processes for human–agent collaboration; measuring impact; realizing value.
3. **AI governance and security** — guardrails, controls, oversight, operational management, lifecycle governance.
4. **Technology and data** — scalable, secure foundations, architectures, and data access patterns.
5. **Organization and culture** — people, roles, incentives, and ways of working.

*(Responsible AI is embedded across all five, not a separate pillar.)*

---

## Quick reference matrix

| Level | AI strategy & experience | Business strategy | Governance & security | Technology & data | Organization & culture |
|-------|--------------------------|-------------------|------------------------|-------------------|------------------------|
| **100 Initial** | No strategy/vision, no exec sponsor; sporadic tactical pilots; no RAI awareness | Human-only, manually intensive workflows; nothing designed for automation | No governance; basic compliance/security; no operating or support model | Fragmented tooling; no reference architectures; limited infrastructure | No training/enablement; no champions or communities; unclear value |
| **200 Repeatable** | Early vision forming; limited leadership alignment; informal strategy | Pilots improve single workflow *steps*; no end-to-end redesign; early value story | Early policies; dev/test/prod separation exists; basic monitoring | Basic environment structure; partial connector reuse | Sporadic training; informal communities; unclear adoption ownership |
| **300 Defined** | Formal agent strategy; cross-functional planning; **exec sponsor**; KPIs tracked and reported | Documented governance model; human–agent teaming defined for priority processes; risks documented and mitigated | Standardized architecture and practices; reusable components; telemetry/data readiness | Formal enablement; defined models and systems; contextual knowledge base | Formal onboarding; active maker communities; regular knowledge sharing |
| **400 Capable** | AI integrated into enterprise planning; cross-department alignment; **RAI guiding design** | Cross-system orchestration; domain redesign; measurable value + optimization loops | Proactive governance with automated monitoring/alerts; **RAI embedded into lifecycle gates** | Scalable enterprise foundations; automated deployment + QA; performance tuning | Champions embedded; shared incentives; culture of optimization |
| **500 Efficient** | AI-first culture; continuous strategic iteration; exec-level accountability | Adaptive, autonomous processes; continuous optimization; **RAI part of enterprise culture** | Predictive risk management; real-time compliance monitoring; automated remediation | Advanced multi-agent patterns; predictive agents guiding reliability | Self-sustaining community; continuous learning with clear incentives |

---

## How to use it
1. **Assess honestly** per pillar — you'll often be at different levels across pillars, and that's the point.
2. **Find the binding constraint** — the lowest pillar usually limits everything else (great tech + no governance = stalled at pilot).
3. **Target one level up**, not five. Progression is sequential.
4. **Gate autonomy on maturity** — don't increase agent autonomy until governance and technology pillars support it.

### Signals you're stuck at 100–200
- Pilots succeed but never reach production.
- Each agent is built differently by whoever built it.
- No named executive sponsor.
- Value is asserted, not measured against a baseline.
- No dev/test/prod separation or DLP strategy.

### What moves you to 300+
An **executive sponsor**, **tracked KPIs with regular reporting**, **documented governance**, **standardized architecture and reusable components**, and **formal enablement with active maker communities**.

---

## Who this is for
Business and technology leaders planning AI adoption · **Centers of Excellence** for AI/Copilot/automation · architects, security leaders, risk professionals · change managers and enablement teams · product owners and transformation leads.

## Related
[plan-scope-and-value.md](plan-scope-and-value.md) (objectives, ROI) · [govern-operate-and-improve.md](govern-operate-and-improve.md) (zoned governance, ALM) · [../reference/metrics-and-roi.md](../reference/metrics-and-roi.md) (the six adoption levers)
