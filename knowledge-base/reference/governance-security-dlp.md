# Reference — Governance, Security & DLP (admin + maker)

Copilot Studio inherits **Power Platform + Microsoft 365** governance. All tenant settings, DLP policies, environment boundaries, Managed Environments, connector restrictions, and ALM pipelines apply to agents. (For maker-facing auth/limits, see [publishing-security-limits.md](publishing-security-limits.md).)

## Security & governance controls (catalog)
| Control | What it does |
|---------|--------------|
| **Agent runtime protection status** | Makers see agent security status on the Agents page. |
| **Data policies (DLP)** | Govern auth, knowledge sources, connectors/actions/skills, HTTP, channels, App Insights, triggers (see below). |
| **Maker audit logs (Microsoft Purview)** | Full admin visibility into maker actions. |
| **Audit logs (Microsoft Sentinel)** | Monitor/alert on agent activity. |
| **Run tools with user credentials** | Tools default to end-user credentials. |
| **Sensitivity labels (SharePoint knowledge)** | Highest label surfaced in responses; per-reference labels in chat. |
| **Certificate auth** | Entra ID manual auth with certificate provider. |
| **Maker security warning / automatic security scan** | Alerts before publishing when secure defaults are modified. |
| **Environment routing** | Route makers into safe build environments. |
| **Maker welcome message** | Inform makers of privacy/compliance requirements. |
| **Autonomous-agent governance via data policies** | Manage trigger capabilities to prevent exfiltration. |
| **CMK** | Customer-managed encryption keys for environments. |
| **Customer Lockbox** | Controlled Microsoft access to customer data (note: doesn't cover Agent 365 security audit logging). |

Plus: **SDL** (Security Development Lifecycle), **geographic data residency**, **compliance certifications** (Trust Center), Product Terms + Data Protection Addendum. Admins can **disable AI-agent publishing**, **disable cross-geo data movement**, and govern which agents appear in M365 Copilot (M365 admin center).

## DLP / Data policies for agents (deep)
Data policies (PPAC → **Security → Data and privacy → Data policy**) classify **Copilot Studio connectors** into **Business / Non-business / Blocked**. **Data can't flow between connectors in different groups.** **Enforcement is real-time** and applies to **all tenants since early 2025** (no more exemptions). When a policy blocks something, **Publish is disabled** and an error banner appears (Channels page → expand error → **Download** the violation spreadsheet).

> New connectors often land in the **Non-business** default group, which many orgs auto-block — e.g. **"Chat without Microsoft Entra ID authentication"** and **"Direct Line channels"**. If a maker is unexpectedly blocked, check the connector's data group.

### Common policy use cases → connector to block
| Goal | Connector name (PPAC) |
|------|------------------------|
| Require user auth (block no-auth) | **Chat without Microsoft Entra ID authentication in Copilot Studio** |
| Block SharePoint/OneDrive knowledge | **Knowledge source with SharePoint and OneDrive in Copilot Studio** (endpoint filtering supported) |
| Block public-website knowledge | **Knowledge source with public websites and data in Copilot Studio** (endpoint filtering) |
| Block document knowledge | **Knowledge source with documents in Copilot Studio** |
| Block connectors as tools | *the specific prebuilt/custom connectors* (also blocks MCP tools that rely on them) |
| Block HTTP node | **HTTP** (endpoint filtering supported) |
| Block skills | **Skills with Copilot Studio** |
| Block Teams/M365 channel | **Microsoft Teams + Microsoft 365 Channel in Copilot Studio** |
| Block Direct Line (demo/custom web/mobile) | **Direct Line channels in Copilot Studio** |
| Block Facebook / SharePoint / WhatsApp / Omnichannel channels | respective **… channel in Copilot Studio** |
| Block event triggers / automated evaluations | **Microsoft Copilot Studio** |
| Govern App Insights connection | **Application Insights in Copilot Studio** |

**Endpoint filtering** (Configure connector → Connector endpoints) allows/denies specific SharePoint sites, websites, or HTTP endpoints instead of blocking the whole connector.

### Configure a policy (steps)
PPAC → Security → Data and privacy → **Data policy** → New/Edit → add environments → **Assign connectors** (⋮ → Block, or Configure connector → endpoints) → **Define scope** (all / multiple / exclude environments; tenant scope covers all) → **Create/Update** → verify in Copilot Studio.

> **If no channel is allowed** (and makers don't configure an allowed one — Direct Line is allowed by default), agents **can't be published** at all.

### Verifying enforcement
Open the agent, attempt the blocked operation → error banner → **Channels page → Download** the violation details (a row per violation). Makers send the spreadsheet to admins to adjust policy, or fix the agent (e.g. switch to **Authenticate with Microsoft/manually**).

### Admin nicety
`Set-PowerAppDlpErrorSettings` (PowerShell) adds an **admin contact email + "Learn more" link** to DLP error messages (tenant-wide).

## Environment strategy (for our framework)
- Maintain **distinct Dev / Test / Prod** environments with **DLP per environment**.
- Use **Managed Environments** for stronger governance; **pipelines** for auditable promotion.
- Discover impacted agents with the **CoE Starter Kit** Power BI dashboard (classic Teams-app chatbots aren't discoverable there — use a Dataverse `List rows` flow).
- Prefer **Authenticate with Microsoft**, **end-user tool credentials**, least-privilege connection references, and **CMK** where required.
