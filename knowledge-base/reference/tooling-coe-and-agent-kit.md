# Reference — Ecosystem Tooling (CoE Starter Kit & Copilot / Power CAT Agent Kit)

Beyond Copilot Studio itself, two Microsoft toolkits massively help **build, test, govern, and optimize** agents at scale. Both are **Dataverse solutions** you install into an environment.

## Copilot Agent Kit (Power CAT Copilot Studio Kit)
Maintained by the **Power Customer Advisory Team (Power CAT)**. GitHub: `microsoft/Power-CAT-Copilot-Studio-Kit`. Docs: *Copilot Agent Kit overview* (guidance/kit-overview). It stores config + test data in **Dataverse** and supports **Excel import/export** for bulk create/update.

### Components (the ones that matter for us)
| Component | What it does |
|-----------|--------------|
| **Test capabilities** | Configure agents, tests, and **test sets**; **batch test** custom agents. Results include **latencies, observed responses, pass/fail**. Test types: **response match, attachment match, topic match, generative answers**. Supports **user-defined rubrics** for generative answers. |
| **Automated testing via Power Platform Pipelines** | Validate agents with test runs **before promotion**; only agents passing the required cases proceed — CI/CD gating for agents. |
| **Rubrics refinement** | Create/test/iterate reusable evaluation rubrics; side-by-side testing, scoring consistency. |
| **Conversation KPIs** | Aggregated performance KPIs complementing built-in analytics. |
| **Agent Insights Hub** | Centralized analytics aggregating **App Insights** + transcripts (usage/quality/trends). |
| **Agent Debugger** | Step-by-step diagnostics of recorded conversations (why the agent answered as it did). |
| **Conversation Analyzer** | Analyze transcripts with **custom prompts** for actionable insights. |
| **Agent Inventory** | Tenant-wide view of all custom agents: features used, auth mode, knowledge sources, metadata. |
| **Agent Review Tool** | Solution analysis for **anti-patterns**/perf/security risks + recommendations. |
| **Compliance Hub** | Define/enforce governance at scale; evaluate configs against controls; thresholds, risk levels, SLA timers, enforcement (review/quarantine/delete); maker remediation via Teams/Outlook. |
| **Power Shield** | Approval workflow for **connector access in DLP** (makers request, admins approve/reject) + compliance questionnaires. |
| **Agent Library** | Discover/install/manage **prebuilt templates + reusable components** (see [agent-types-and-m365-extensibility.md](agent-types-and-m365-extensibility.md)). |
| **SharePoint synchronization** | Periodically sync selected SharePoint content into agent knowledge as files. |
| **Prompt Advisor** | Score/analyze/optimize prompts. |
| **Webchat Playground** | Customize Web Chat appearance (colors/fonts/thumbnails). |
| **Adaptive Cards Gallery** | Ready-made Adaptive Card templates + interaction examples. |
| **Agent Value** | Classify agents by type/behavior/value; align to strategy; measure impact. |

### Why it matters for our framework
- **Batch testing + rubrics** = a real regression suite beyond the built-in Evaluate tab (see [analytics-evaluation.md](analytics-evaluation.md)); wire it into **pipelines** for gated promotion.
- **Agent Inventory + Agent Review Tool + Compliance Hub** = governance and anti-pattern detection at scale — complements our conventions ([../11-code-first-framework-and-workflow.md](../11-code-first-framework-and-workflow.md)).
- **Agent Debugger + Conversation Analyzer + Insights Hub** = deep post-hoc diagnosis when the activity map isn't enough.

## CoE Starter Kit (Center of Excellence)
A broader Power Platform governance toolkit (Power BI dashboards + flows). For Copilot Studio it provides a **tenant-wide overview of agents and environments** — used to **discover agents affected by DLP** and to run maker remediation campaigns ([governance-security-dlp.md](governance-security-dlp.md)).

> Caveat: **classic chatbots** created with the legacy Teams-app aren't discoverable in CoE — use a Dataverse **List rows** flow to enumerate all agents/chatbots in an environment.

## How this fits our build workflow
1. Build code-first in VS Code (clone/edit/sync).
2. **Agent Review Tool** for anti-patterns before promotion.
3. **Kit test sets + rubrics** as the regression suite; **pipeline** gates dev→test→prod.
4. **Agent Inventory / Compliance Hub / Power Shield** for governance; **CoE** for tenant discovery.
5. **Insights Hub / Conversation Analyzer / Agent Debugger** for production optimization.
