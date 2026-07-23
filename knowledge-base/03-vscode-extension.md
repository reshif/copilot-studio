# 03 — The Copilot Studio Extension for VS Code

**Status: Generally Available (GA) since 2026-01-14.** Monthly releases. Issues/roadmap: <https://github.com/microsoft/vscode-copilotstudio/issues>.

This extension brings enterprise agent authoring into VS Code, bridging cloud-based Copilot Studio and local development. You work directly with **cloud agents** using local tools, Git, and AI assistants.

## Why use it (the pitch)

- **Developer-friendly**: YAML syntax highlighting, **IntelliSense** completion, keyboard shortcuts, full-text search, fast navigation.
- **Local dev loop**: clone the full agent definition to disk, edit in YAML (or via an AI agent), **apply** to your environment, preview/test in-product.
- **Git-based collaboration**: version agent definitions, PR review, change history/auditability — using your existing team workflows. Crucially, you get the **full agent definition**, not just the solution file.
- **AI-assisted authoring**: use **GitHub Copilot, Claude Code, or any VS Code AI assistant** to draft topics, update tools, fix issues, then sync back.

## Prerequisites

**System**
- OS: Windows x64 (Win 10 1809+ or Win 11) **or** macOS.
- VS Code **1.80+** (latest stable recommended).
- Internet connection (install + auth).

**Account**
- Microsoft account with a **Copilot Studio license/access**.
- Permission to access **at least one** Copilot Studio environment.
- **Read & write** access to any agents you plan to edit.

## Install

**Method 1 — from VS Code**: Extensions pane (`Ctrl+Shift+X`) → search "Copilot Studio" → confirm publisher is **Microsoft** → **Install** → **Reload** if prompted. A Copilot Studio icon appears in the activity bar.

**Method 2 — Marketplace site**: `https://marketplace.visualstudio.com/items?itemName=ms-CopilotStudio.vscode-copilotstudio` → **Install** → allow it to open VS Code.

## Authenticate & first launch

1. Click the Copilot Studio activity-bar icon → the extension pane prompts you to **Sign In** → **Allow**. Browser opens Microsoft auth.
2. Complete credentials + MFA.
3. Approve the requested permissions: **Read & write Copilot Studio agents**, **Access environment information**, **Sync files to cloud** → **Accept/Allow**.
4. Return to VS Code. The **Agents** pane loads your environments and their agents, in a tree:

```text
AGENTS
  └─ Environment Group (developer, default, sandbox, production, ...)
      ├─ Environment 1
      │   ├─ Agent 1
      │   └─ Agent 2
      └─ Environment 2
          └─ Agent 3
```

## Key capabilities (feature map)

| Feature | Description |
|---------|-------------|
| **Agent cloning** | Download an existing agent's full definition to a local workspace. |
| **YAML editing** | Edit components as structured YAML with IntelliSense. |
| **Component management** | Modify knowledge sources, knowledge files, tools, topics, triggers, skills. |
| **Sync operations** | Preview / Get / Apply changes between local and cloud. |
| **Environment apply** | Create a **new** agent in a Dataverse environment, or **update** an existing one. |

## The core workflow (clone → edit → sync)

### 1. Clone
Right-click an agent in the **Agents** pane → **Clone agent** → pick a folder → wait ~10–30s → "Agent Cloned successfully." Details, folder layout, and alternate methods (command palette, from agent URL) are in [04-agent-definition-yaml-schema.md](04-agent-definition-yaml-schema.md).

- **Command palette**: `Ctrl+Shift+P` → `Copilot Studio: Clone Agent`.
- **From URL**: copy the agent URL in the portal (**Settings → Agent details**, format `https://copilotstudio.microsoft.com/environments/{guid}/bots/{guid}`); the extension detects it on your clipboard and marks the agent "(from clipboard)".
- **Reattach**: `Copilot Studio: Reattach Agent` connects a local folder (e.g. cloned from a teammate's Git repo, or to point at a different environment) back to Copilot Studio.

### 2. Edit
Edit YAML with IntelliSense (`Ctrl+Space` for context-aware suggestions, `Ctrl+F` search). Problems show in the **Problems** pane (`Ctrl+Shift+M`) and as red underlines. Changed/saved files show in a different color. Full component detail in [05](05-topics-and-dialogs.md)–[08](08-power-fx-and-variables.md).

### 3. Sync — three operations

| Operation | Direction | Effect / safety |
|-----------|-----------|-----------------|
| **Preview** | Cloud → Local | Shows remote changes **without** applying. No local change. |
| **Get** | Cloud → Local | Downloads & applies remote changes to local files. On same-component conflicts, shows both versions; you confirm which to keep. May overwrite uncommitted local changes (prompts). |
| **Apply** | Local → Cloud | Uploads local changes; **modifies the live agent immediately**. **Not** a publish. Blocked if there are un-**Get**-ten remote changes. |

**Apply ≠ Publish.** Apply updates the agent definition in the environment so you can **test in the Copilot Studio test pane**; publishing to channels is a separate step (see [10](10-alm-publishing-testing.md)).

**Agent Changes pane** shows Local Changes and Remote Changes per agent:
```text
AGENT CHANGES
├─ Agent 1
│  ├─ Local Changes (1)
│  │  └─ topics/greeting.topic.yaml
│  └─ Remote Changes (1)
│     └─ agent.yaml
```

**Command-palette equivalents**: `Copilot Studio: Preview`, `Copilot Studio: Get Changes`, `Copilot Studio: Apply Changes`, `Copilot Studio: Focus on Agents View`.

### Before you Apply — checklist
- No unresolved merge conflicts.
- Ran **Preview**/**Get** for latest remote changes.
- Files pass validation (no errors in Problems pane).
- Committed to Git (if using version control).
- You have permission to modify the agent.

## Useful keyboard shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+Shift+X` | Extensions pane |
| `Ctrl+Shift+P` | Command palette (all `Copilot Studio:` commands) |
| `Ctrl+Space` | Context-aware IntelliSense suggestions |
| `Ctrl+F` | Search variable names / values across the agent |
| `Ctrl+Shift+M` | Problems pane |

## Troubleshooting (common)

- **Icon missing** → reload VS Code, ensure extension isn't disabled, update VS Code, reinstall.
- **Auth fails/loops** → clear `microsoft.com` cache/cookies, try another browser as default, disable blocking browser extensions, check corporate firewall/proxy, use a supported account type.
- **Can't find extension** → search in the Extensions pane (not web), check spelling, ask admin whether Marketplace access is restricted.
- **Uninstall** → Extensions pane → Copilot Studio → gear **Manage → Uninstall** → reload. *Local cloned files are NOT deleted; remove manually.*

## Limitations / gotchas to remember

- **Apply is immediate and live** — it changes the real agent in that environment (but doesn't publish).
- **Apply is blocked** until you Get pending remote changes — always Preview/Get first when collaborating.
- For **multi-environment** promotion (dev→test→prod), prefer **solutions + pipelines**, not repeated cloning/reattaching (see [10](10-alm-publishing-testing.md)).
- The simplest way to start a brand-new agent today: **create a basic agent in the portal, then clone it**. (Microsoft is adding from-scratch creation in the extension.)

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
