# Reference — Computer Use (CUA)

**Computer use** lets an agent operate a **Windows machine's GUI** — websites and desktop apps — via a virtual mouse/keyboard, driven by natural-language instructions. Powered by **Computer-Using Agents (CUA)**: vision + reasoning models that adapt to UI changes (no brittle selectors, no code). Use it when there's **no API** — data entry, invoice processing, data extraction.

> **Requires generative orchestration ON.** Add as a tool: Tools → Add tool → New tool → **Computer use**.

## Configure (4 required fields)
- **Name**, **Description** (drives when the agent uses it).
- **Model**:

| Provider | Model | Tier | Status | Credits/step |
|----------|-------|------|--------|--------------|
| OpenAI | Computer-Using Agent (CUA) | Standard | GA | 5 |
| Anthropic | Claude Sonnet 4.5 | Standard | GA | 5 |
| Anthropic | Claude Sonnet 4.6 | Standard | Experimental | 5 |
| Anthropic | Claude Opus 4.6 | Premium | Experimental | 15 |

(Anthropic models require the admin to enable **access to external models** for the environment.)
- **Instructions** — the step-by-step task (URLs, app names).

## Optional settings
- **Inputs** — dynamic values combined with instructions at run time (e.g. per-run form values).
- **Machine** — target machine (managed in Power Automate; Refresh / Manage machines / See details).
- **Connection** / **Credentials to use**:
  - **Maker-provided** (default) — runs as the author; suits autonomous agents. ⚠️ If you share the agent, users act with the **author's** access on that machine.
  - **End-user** — each user needs machine credentials.
- **Human supervision** — email (Outlook) reviewer contacted if potentially harmful instructions are detected; a response-time limit; run expires/stops if no response. Reviewer should be the run initiator (they see the activity).
- **Stored credentials** — for site/app sign-ins during a run: **internal storage** (Power Platform encrypts) or **Azure Key Vault**. Provide type (Website/Desktop app), username, password/secret, and login domain (`*.contoso.com` wildcards) or desktop app/process name (`msedge`, `Notepad`). Password fields unsupported on some app types (Electron/Java/Unity/games/CLI/Citrix/virtualized).
- **Access control** — restrict to an **allow list** of URLs/apps. Note: it blocks *actions* on non-allowed sites, not *opening* them.
- **Enforce HTTPS** — only interact with `https://` sites.

## Billing
Bills at the **Agent action** rate: **5 credits/step** (standard) or **15 credits/step** (premium model). A "step" may bundle several low-level clicks/keystrokes. Example: a 4-step timesheet run = **20 credits** (standard) / **60 credits** (premium). **CUA is NOT included** in the M365 Copilot USL.

## Testing & publishing
- **Test**: left panel = instructions + reasoning/action log; right = live machine preview; **Stop testing** halts immediately.
- **Best for autonomous agents** (background tasks). In conversational use, user auth means each user needs machine credentials, and the chat shows reasoning + screenshots.

## Best practices
**Instructions** (like briefing a colleague): full URLs + exact app names; state actions explicitly ("select **Submit**, no confirmation needed"); break down complex UI; step-by-step lists for long tasks.
**Extraction**: describe what to extract in the CUA instructions; return as **text** or **JSON** (specify the schema); to act on it (e.g. email), add both the CUA tool and the email tool and say so in the **agent** instructions.
**Secure the machine**: dedicated/isolated machines; least-privilege account; allow-list trusted websites (e.g. Edge policy via Intune); allow only essential apps (App Control).
