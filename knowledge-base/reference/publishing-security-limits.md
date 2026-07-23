# Reference — Publishing, Channels, Security & Limits (complete)

## Part 1 — Publishing & channels

**Publish** makes the current agent version available to end users on **all connected channels**. You must publish before customers can use the agent, and **re-publish after every change**.

### Publish flow
Open agent → **Publish** → confirm (takes minutes). New content only reaches users on a **new session** (most channels end a session after **30 min** inactivity). In persistent channels (Teams, Omnichannel) type **`start over`** to force the latest version immediately; otherwise it can take **up to 1 hour** to take effect.

### Test vs demo vs production
- **Test agent panel** — while building; verify flows and spot errors (also shows the activity map).
- **Demo website** — for teammates/stakeholders (No-auth or manual-auth agents); **not for production/customers**.
- **Publish for yourself first**, validate, then **Make the agent available to others**.

### Channels (configure after first publish: **Channels** menu)
- **Teams and Microsoft 365 Copilot**
- **SharePoint**
- **WhatsApp** (since July 2025)
- **Demo Website** / **Custom Website** (embed)
- **Mobile App** (custom application)
- **Facebook**
- **Azure Bot Service channels**: Cortana, Slack, Telegram, Twilio, Line, Kik, GroupMe, Direct Line Speech, Email
- **DirectLine** API (custom apps) / telephony (voice)

### Channel experience differences (selected)
- **CSAT survey**: Adaptive card on Website; **text-only** on Teams/Facebook/Omnichannel.
- **Multiple-choice**: Teams renders up to **6** (hero card); Facebook up to 13.
- **Markdown**: full on Website; partial on Teams/Facebook/Omnichannel.
- **Attachments**: users **can't upload** attachments in any channel (agent replies it only processes text) unless routed to a skill that handles them.

### Troubleshooting publish
Verify settings/auth/channel config; check dependencies (topics, flows, connectors, data sources); read the **Publish page** status/error codes.

---

## Part 2 — Security & authentication

Agents default to **Authenticate with Microsoft** (Entra ID) — auto-SSO for Teams, Power Apps, M365 Copilot with no manual setup.

### Three auth options (Settings → Security → Authentication)
| Option | Behavior |
|--------|----------|
| **No authentication** | Anyone with the link can chat. **Can't use tools with user credentials.** Not recommended for org/internal use. |
| **Authenticate with Microsoft** (default) | Entra ID SSO; user variables available; **no** `User.AccessToken`; can't add explicit Authenticate nodes. |
| **Authenticate manually** | Generic OAuth2 or Entra ID; enables **Sign in** system topic and **Authenticate** nodes; provides `User.AccessToken`. Optionally **Require users to sign in**. |

### Sign-in mechanisms
- **Sign in** system topic — triggers at conversation start when manual auth + "Require users to sign in" is on. Keep it to auth only (don't add other actions).
- **Authenticate** node — add to a custom topic to sign in mid-conversation. **Leaf node only**; auto-creates **success** (`User.IsLoggedIn = true`) and **failure** paths. Once signed in, not prompted again. Passing `User.AccessToken` before an Authenticate node prompts sign-in at that point.
- **SSO** with Entra ID means users aren't prompted at all.
- Always author **both** success and failure paths.

### Auth variables
See [variables-and-entities.md](variables-and-entities.md#authentication-variables). `User.DisplayName`←`name` claim (needs `profile` scope); `User.Id`←`sub` claim. **Guard `User.AccessToken`** — never in Message nodes or untrusted flows.

### Tool auth
Tools run in the **user context** and need auth enabled. **End-user** vs **Maker-provided** credentials (see [tools.md](tools.md)). No-auth agents can't run credentialed tools.

### Governance (broader)
Apply DLP policies, environment strategy, and the platform's **security and governance** controls; combine auth with least-privilege connection references. Sensitivity-labeled (*confidential*/*highly confidential*) and password-protected docs can't be indexed as knowledge.

---

## Part 3 — Quotas & limits

### Quotas (requests per minute, per Dataverse environment)
- **8,000 RPM** — messages to an agent (any user/integration message).
- **Generative AI messages** (orchestration, actions, AI tools, agent flow actions, generative answers), by prepaid message packs:
  - 1–10 packs: **50 RPM / 1,000 RPH**
  - 11–50 packs: **80 RPM / 1,600 RPH**
  - 51–150 packs: **100 RPM / 2,000 RPH**
  - +10 packs above 150: **+1 RPM / +20 RPH**
  - Trial/dev env: **10 RPM / 200 RPH**
  - Pay-as-you-go / M365 Copilot users: **100 RPM / 2,000 RPH**
- At quota, the user sees a failure notice.

### Web app limits
| Feature | Limit |
|---------|-------|
| Knowledge sources per agent | **500** (all types) |
| Instructions (Copilot agent) | **8,000 characters** |
| Connector payload | 5 MB (public cloud); 450 KB (GCC) |
| File upload size | **512 MB** |
| Files uploaded (count) | **500** |
| Image upload | Only inside PDFs |
| Skills | **100** per agent |
| Topics | **1,000** per agent (Dataverse env) |
| Trigger phrases | **200** per topic |
| Tools (orchestrator max) | **128** (recommend ≤25–30) |

### Teams-app limits
Agents **50**/team; skills **100**/agent (needs standalone subscription); topics **250** (Dataverse-for-Teams) / **1,000** (after upgrade); trigger phrases **200**/topic.

### Subscription limits
- Standard: Power Platform requests **250,000 / 24h**.
- Teams (select M365): **10 sessions/user/24h**; **6,000 requests/24h**.
- Power Automate flows triggered by the agent consume Power Platform requests.

### SharePoint knowledge limits (integrated)
25 site URLs (generative); modern pages only (no SPFx/classic ASPX); DOC/DOCX/PPT/PPTX/PDF; <7 MB files without in-tenant M365 Copilot license; lists ≤15 (≤35k rows/list, ≤120k total; queries read first 2,048 rows; ≤12 lookup columns in default view); omit `https://`; no metadata/folder-count queries.

### OneDrive / uploaded-SharePoint knowledge limits
1,000 files, 50 folders, 10 subfolder levels, 512 MB/file; 4–6h sync; DOC/DOCX/XLS/XLSX/PPT/PPTX/PDF; **no** confidential/password-protected docs; no glossaries/synonyms; **ALM not supported**. Salesforce/Confluence/ServiceNow/Zendesk: no article count/size limit, 4–6h sync, ALM not supported.

### Dataverse knowledge limits
2 sources/agent; 15 tables/source; standard/activity tables; READ permission; synonym/glossary name ≤100 chars, description ≤1,000 chars.

### Required network services (don't block)
`*.directline.botframework.com` (HTTPS+WS), `*.powerva.microsoft.com`, `*.analysis.windows.net`, `bot-framework.azureedge.net`, `cci-prod-botdesigner.azureedge.net` (all required); `token.botframework.com` (manual auth only); plus Power Automate required services. Omnichannel (ACS) channel-data message size limit **28 KB** — clear large variables before handoff to avoid `MessageSizeExceeded`.
