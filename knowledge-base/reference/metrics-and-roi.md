# Reference — Metrics Library & ROI Formulas

The shared vocabulary for proving agent value. Use **one canonical name** per metric across all agents so "engagement rate" means the same thing everywhere — that's what turns many separate measurement efforts into one coherent value story.

---

## The four value drivers (organize everything by these)

| Value driver | What it measures | How to price it |
|--------------|------------------|-----------------|
| **Efficiency** | Productive hours returned, reinvested in higher-value work | Productive hours returned × fully loaded productive-hour value |
| **Quality** | Error reduction, consistency, compliance | (Error rate before − after) × volume × cost per error |
| **Revenue** | Retained, expanded, or new business | Conversion/deflection delta × volume × unit revenue × attribution discount |
| **Strategic** | Decision velocity, confidence, optionality, resilience | Option premium + talent retention value + resilience value |

> Organizations that use AI to **change what they measure** — not just improve existing KPIs — see stronger financial results. That's why the Strategic driver is foundational.

### Three traps that weaken the value story after launch
1. **Measurement that stops at pilot** — instrumentation drifts in production. Fix: embed measurement in the deployment workflow.
2. **Activity that doesn't tie to outcomes** — sessions and user counts are *usage*, not value. Every KPI must trace to a value driver.
3. **The time-savings trap** — claiming value from theoretical time savings alone destroys credibility. Build a chain of evidence: adoption → operational KPIs → business outcomes.

---

## The Agent Assisted Hours (AAH) formula

**Conversational agents**
```
AAH = (Knowledge references + Weighted sessions without knowledge references)
      × Time savings multiplier ÷ 60
```
- Each knowledge-source reference counts once.
- Sessions **without** references are weighted by outcome: **resolved = 1.0**, **escalated/abandoned = 0.7**.
- **Default time savings multiplier = 6 minutes** per knowledge reference.

**Autonomous agents**
```
AAH = (Knowledge references × retrieval savings
      + Action time savings
      + Successful sessions without actions × generic savings) ÷ 60
```
All multipliers are customizable in the Copilot Studio agents report calculator.

**Agent Assisted Value (AAV)**
```
AAV = AAH × Hourly rate      (default $72/hr, US BLS employer-cost data)
```

### Worked example (customer-service agent, 10,000 engaged sessions/month)
5,000 sessions cite knowledge (avg 2 references each); the other 5,000 split into 3,000 resolved and 2,000 escalated/abandoned.

1. Knowledge references: `5,000 × 2 = 10,000`
2. Weighted non-citing sessions: `(3,000 × 1.0) + (2,000 × 0.7) = 4,400`
3. Total: `10,000 + 4,400 = 14,400`
4. Hours: `14,400 × 6 ÷ 60 = ` **1,440 hours/month**
5. Value: `1,440 × $72 = ` **$103,680/month ≈ $1.24M/year**

> Microsoft publishes **16 use-case blueprints** (HR, IT, finance, legal, field service…) each with a worked calculation.

---

## Leading vs lagging indicators (include one of each per driver)

| Driver | Leading | Lagging |
|--------|---------|---------|
| Efficiency | Routine adoption rate at 30 days; theme coverage of top 10 questions | Annualized ROI; cost per transaction |
| Quality | Rubric groundedness score in testing; thumbs-down trend | Error-rate delta vs baseline; audit findings avoided |
| Revenue | Containment + FCR trend; advisor meetings/day | Retention delta; pipeline conversion lift |
| Strategic | Employee sentiment on AI; redesigned workflows in flight | New capabilities shipped; EBIT contribution |

---

## Metrics library

### Engagement & adoption — *are people using it?*
| Metric | Definition | Where |
|--------|-----------|-------|
| **Total sessions** | Analytics sessions in a period (one conversation can produce several) | CS Analytics |
| **Engagement rate** | Share of sessions that triggered a custom topic, Escalate, Fallback, or Conversational Boosting | CS Analytics |
| **Routine adoption rate** | Share of eligible users using the agent on **4+ active days in a rolling 4-week window** (Microsoft's habit threshold) | Copilot Dashboard (Viva Insights) |
| **Per-user active days** | Distinct days a user interacted — separates repeat from one-time users | Copilot Dashboard |
| **Trigger use (autonomous)** | Count + outcome per event trigger — shows which triggers matter and which are noise | Autonomous analytics |

### Outcomes — *are users achieving their goals?*
| Metric | Definition |
|--------|-----------|
| **Resolution rate** | Engaged sessions ending resolved (confirmed or implied) |
| **Escalation rate** | Engaged sessions handed off via Escalate/Transfer |
| **Abandon rate** | Engaged sessions ending after **60 min** inactivity without resolution/escalation — the *fail-quietly* signal |
| **Deflection rate** | Requests self-served rather than escalated — the key tier-1 metric |
| **First-contact resolution (FCR)** | Resolved on first interaction, no return contact **within 7 days** |
| **Run outcomes (autonomous)** | Success/failure + average duration (the autonomous equivalent of resolution rate) |
| **Tool use (autonomous)** | Count + success rate per tool invoked |
| **Knowledge source use** | References per source — shows which content carries the agent and which sits unused |

### Quality & groundedness — *is the output correct, on-brand, safe?*
| Metric | Definition |
|--------|-----------|
| **Generated answer rate** | Share of questions that got a generative answer vs fallback/unanswered |
| **Generated answer quality** | Evaluation score vs reference answers or rubric |
| **Groundedness** | Is the answer supported by cited knowledge (AI judge + configurable rubric) |
| **Instruction-following score** | Did it follow system/user instructions |
| **Topic match score** | Did the expected topic trigger — your intent-recognition regression signal |
| **Citation accuracy** | Share of claims actually supported by attached citations (sampled review) |
| **AI-generated insights** (preview) | Daily recommendations on unanswered questions with coverage fixes |

### Voice-of-customer & qualitative
**CSAT** (1–5 from End of Conversation survey) · **Reactions** (thumbs up/down + comments, retained **28 days**) · **Sentiment** (preview — % of sessions with negative sentiment) · **Themes** (preview — AI-clustered user questions, convertible to test sets in one step) · **Customer-experience narrative** (transcript themes beyond CSAT) · **User confidence** · **Manager stories of reclaimed capacity** (structured quarterly interviews) · **Employee sentiment on AI** (Viva Glint) · **Talent attraction/retention signal**.

### Productivity & process
**Cycle time** (report median, **P90**, **P99** — the tail matters more than the median) · **Average handle time (AHT)** · **Touchless rate** (autonomous end-to-end, no human) · **Time to first answer** · **MTTD/MTTR** (security) · **First-time fix rate** (field service).

### Productive-hour value
**Agent Assisted Hours** · **Agent Assisted Value** · **Time savings multiplier** (override in the calculator) · **Monthly savings per run** and **per tool** (entered by the owner on the Analytics page; updates retroactively) · **Custom metrics** (preview — up to **3 maker-defined metrics per agent** described in natural language, scored on a session sample, shown as labeled donuts) · **Cost per transaction** (your baseline).

### Business outcomes — *did the business numbers move?*
**Conversion lift** · **Retention delta** · **Cross-sell rate** · **Forecast accuracy** · **DSO delta** · **Off-catalog spend share** · **Audit findings avoided**. The agent doesn't produce these directly — it influences them through better service, faster cycles, and more productive hours. Read via the Copilot business impact report joined to your ERP/CRM.

---

## Monthly optimization review (what to look at)
Deflection rate · Resolution rate · Engagement rate · **Topics with low resolution** · **Unrecognized utterances** · **Per-channel analysis** (performance differs by channel).

## Six adoption levers (the license isn't the investment — adoption is)
Uneven ROI usually means gaps in **training, role modeling, or workflow alignment** — not technology. In order of impact:
1. **Structured, role-based training as a prerequisite** — train **managers first** so they lead the change.
2. **Anchor every agent to a named, high-volume workflow** — deflecting a specific Tier-1 task shows ROI faster than generic productivity.
3. **Executives model adoption** — visible executive usage is the *single strongest accelerator*.
4. **Shared prompt libraries**, updated monthly from Analytics themes.
5. **Visible guardrails** — users trust governance they can see.
6. **Quarterly expansion rhythm** — pick a workflow, build, measure against baseline for **90 days**, review with the sponsor, then scale or retire.

## Custom analytics architecture
Copilot Studio analytics come from an internal service; usage is also written to the Dataverse **`conversationtranscript`** table. **Default retention is 30 days** (changeable). Key tables: `bot`, `botcomponent`, `conversationtranscript` (the large one).

For longer retention or custom dashboards, export to **Azure Data Lake Storage Gen2 via Azure Synapse Link for Dataverse** (incremental sync, Common Data Model format; select `ConversationTranscript` — the other two don't support incremental sync).

> ⚠️ **Synapse Link mirrors deletes.** The recurring bulk-delete job that purges transcripts older than 30 days will also remove them from the lake. Use **snapshots** or configure **append-only mode**.

Also available: **Copilot Agent Kit Conversation KPIs** (aggregated metrics in Dataverse, long-term retention, custom variables like NPS, full transcripts alongside KPI records, extensible Power BI reports).
