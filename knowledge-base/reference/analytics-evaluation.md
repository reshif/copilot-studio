# Reference — Analytics, Monitoring & Evaluation

Two feedback loops: **Analytics** (what happened in production) and **Evaluation** (systematic quality testing before/after changes).

---

## Part 1 — Analytics

**Analytics** page in Copilot Studio. Tailored views for **conversational** vs **autonomous** (trigger) agents, plus a **hybrid** view. Data retention: metrics **360 days**; session details & transcripts **28–29 days**. Times in **UTC**. **Test-panel activity is excluded.**

### Access & sharing
Share the **Analytics Viewer** role (individuals only, not groups) for view-only analytics; add **Bot Transcript Viewer** security role for transcript access.

### Summary & overview
- **Summary** — Copilot-generated AI insights (themes, metrics, trends, sentiment); **customer comments summary** (preview) groups feedback.
- **Active users** — DAU/MAU (unique users who interacted ≥once). **Requires authentication enabled.** Avg DAU = sum of full-day counts ÷ full days.

### Sessions & conversations
- A **conversation** = ongoing interaction on a channel; can pause/resume or transfer to a rep. **Times out after 30 min** inactivity (telephony: 3 min after End Conversation). One conversation can contain multiple **analytics sessions**.
- **Classic mode** session = the last custom topic triggered (else last system topic).
- **Autonomous** session = from trigger payload received through the actions taken. Only **successful** trigger runs are tracked.

### Key metric areas
- **Effectiveness** (conversational): engagement, **conversation outcomes** (`Resolved` / `Escalated` / `Abandoned`), resolution/escalation rates.
- **Satisfaction**: **CSAT** from end-of-conversation surveys.
- **Use**: Conversations vs Runs (hybrid: **All** shows both).
- **Autonomous health**: run outcomes, **knowledge source use**.
- **Themes**: clustered topics of what users ask (use instead of topic analytics under generative orchestration).
- **Topic analytics** (classic only): per-topic outcomes, total use trend, CSAT + trend. Access via Topics page → **… → Analytics**.

### Transcripts
Downloadable a few minutes after a conversation times out; any window in the last **29 days**; via **Dataverse (Power Apps portal)** or **session chat transcripts (Copilot Studio app)**. Up to ~1h before a session appears on the dashboard. Teams app can't download Dataverse transcripts (use the web app).

### Telemetry / Application Insights
Connect agents to **Application Insights** for deep telemetry (governable via DLP — the "Application Insights in Copilot Studio" connector). ROI/business-value guidance: see Microsoft's *Measure the ROI and business value of AI agents*.

> **M365 Copilot agents** (declarative) do **not** feed the Copilot Studio Analytics page.

---

## Part 2 — Evaluation (test sets)

Build **test sets** (Evaluate tab / evaluation feature) of test cases and run **test methods** against them. A test set is **single response** or **conversation** type. All start with **General quality** by default. Automated evaluations can be blocked/limited by DLP (the "Microsoft Copilot Studio" connector).

### Test methods

| Method | Measures | Set type | Scoring | Needs |
|--------|----------|----------|---------|-------|
| **General quality** | Overall answer quality via LLM | single or conversation | /100% | none |
| **Compare meaning** | Intent/meaning match to expected | single | /100% | pass score, expected answer |
| **Tool use** | Used the expected tools/topics | single | pass/fail | expected capabilities |
| **Keyword match** | Contains expected keywords (Any/All) | single or conversation | pass/fail | expected keywords |
| **Text similarity** | Wording/structure match (cosine 0–1) | single | /100% | pass score, expected answer |
| **Exact match** | Character-for-character identical | single | pass/fail | expected answer |
| **Custom** | Your criteria via labels | single or conversation | pass/fail | name, instructions, labels |

### General quality criteria (LLM-scored)
Must meet **all**: **Relevance** (on-topic, answers the question), **Groundedness** (based on provided context, not invented), **Completeness** (all necessary info), **Abstention** (did it attempt to answer). Missing one → flagged. No expected answer needed. *Note: fewer knowledge sources doesn't guarantee better grading (retrieved context may still be large).*

### Method-picking guide
- **No single correct answer** → General quality and/or Compare meaning.
- **Must use a specific tool/topic** → Tool use.
- **Must contain terms** → Keyword match (Any/All).
- **Must match wording** (legal/precise) → Text similarity (+ Compare meaning for meaning).
- **Exact short value** (code/number) → Exact match.
- **Domain policy (e.g. HR compliance)** → Custom: write goal-oriented **evaluation instructions** + **labels** (e.g. Compliant/Non-Compliant) each mapped to Pass/Fail.

### Custom method example
```md
Evaluate the agent's response for HR policy compliance.
What to check:
- Protects privacy; doesn't reveal/request sensitive data.
- Avoids discrimination, bias, inappropriate judgments.
- Provides safe, neutral, HR-aligned guidance.
- Doesn't give legal advice or definitive claims.
```
Labels: `Compliant` (Pass), `Non-Compliant` (Fail).

### Workflow
Create/edit test set → **Add test method** (multiple allowed; set pass scores/criteria) → add expected answers/keywords/tools per case (missing expected data ⇒ *Invalid* result) → **Save** → run → use results to iterate. Pair with the **activity map** (test panel) to debug routing and with production **Analytics** to catch regressions.
