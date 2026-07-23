# 10 — ALM, Publishing & Testing

This is how an agent goes from "edited locally" to "running in production, safely and repeatably."

## Environments & solutions (the ALM backbone)

- Agents live in a **Dataverse environment** (dev / test / prod / sandbox / default). The extension shows environments grouped in the **Agents** pane.
- **Solutions** are the Power Platform packaging unit used to move agents (and their dependencies: flows, connection references, environment variables) between environments.
- **Recommended promotion path**: use **solutions + pipelines** (auditable, scalable) rather than repeatedly cloning/reattaching across environments.

### Two promotion approaches

1. **Solution-based (recommended for prod)** — package the agent in a solution; export/import or use **Power Platform Pipelines** / DevOps to promote dev → test → prod. Handles connection references and environment variables properly.
2. **Extension reattach (dev/testing convenience)** — the VS Code extension can synchronize a local agent definition to *another* environment via **`Copilot Studio: Reattach Agent`**, then test there. Use for quick loops; still layer org processes (auditing, telemetry) for real shipping.

## The Apply vs Publish distinction (critical)

| Action | Where | Effect |
|--------|-------|--------|
| **Apply** (VS Code) | Local → environment | Updates the **agent definition** live in that environment so you can test in the Copilot Studio test pane. **Does not publish.** |
| **Publish** (Copilot Studio) | Environment → channels | Makes the current agent version **available to end users** on its channels. |

So the loop is: **edit → Apply → test in test pane → (when ready) Publish**.

## Git-based collaboration (with the extension)

- **Solo**: clone → `git init` from Source Control → make changes/commits → Apply when ready.
- **Team**: first dev clones → `git init` → pushes to shared repo (GitHub/Azure DevOps) → others clone from Git → each runs **`Copilot Studio: Reattach Agent`** to connect their local folder to Copilot Studio → collaborate via Git (PRs, history) → each applies to an environment or pushes downstream per their SDLC/pipelines.
- Best practices: init Git right after cloning; meaningful folder names; back up the clone (GitHub/ADO); document clone locations. Don't clone to temp dirs; don't clone multiple times to different locations.

## Publishing & channels

After publishing, an agent can be surfaced on channels such as:

- **Microsoft Teams**
- **Custom website / web chat** (and **DirectLine** for custom apps)
- **Microsoft 365 Copilot**
- **WhatsApp** (from July 2025)
- Others (varies by release)

> Reminder: **Fabric Data agents don't function when the main agent is deployed to Microsoft 365 Copilot** (see [09](09-multi-agent-orchestration.md)).

## Testing & evaluation

**Interactive testing**
- **Copilot Studio test pane** (web) — after **Apply**, chat with the agent; for autonomous flows, fire the trigger. The **activity map** shows routing (which topic/tool/agent handled a turn) — invaluable for debugging orchestration.
- **New experience → Preview tab** — interactive preview chat.

**Systematic evaluation**
- **New experience → Evaluate tab** — create and run **test sets** to measure quality.
- Code-first testing options (from the `skills-for-copilot-studio` framework, see [11](11-code-first-framework-and-workflow.md)):
  - `chat-with-agent` — send utterances via the **Copilot Studio Client SDK** (published agents only).
  - `run-tests` — batch testing ("Kit mode" via Dataverse API) or offline eval analysis ("Eval mode" from CSV exports).
  - `directline-chat` — test via **DirectLine v3** REST API (supports OAuth).

**Monitor (post-deploy)**
- **Monitor tab** (new experience) — recent tasks, files accessed, activity.
- Analytics/telemetry for conversation outcomes (`conversationOutcome` on nodes feeds this).

## A repeatable release checklist

1. Edit locally; validate (no Problems-pane errors; `=`-prefixed Power Fx; unique IDs).
2. Commit to Git; open PR; review.
3. **Preview/Get** remote changes; resolve conflicts.
4. **Apply** to **dev**; test in test pane (check activity map).
5. Run **evaluation** test sets; fix regressions.
6. Package in a **solution**; promote dev → test → **prod** via pipeline (reconnect connection references / set environment variables per env).
7. **Publish** to channels in prod.
8. **Monitor**; iterate.

## ALM gotchas to remember

- **Connection references** and **environment variables** are environment-specific — solutions/pipelines handle rebinding; ad-hoc reattach does not fully.
- **Apply is immediate + live** in the target environment (but not published) — don't Apply straight to prod casually.
- Prefer solutions for anything auditable/customer-facing; reserve reattach for fast inner-loop dev/testing.

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
