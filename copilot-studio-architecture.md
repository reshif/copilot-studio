# Copilot Studio — Enterprise Architecture (Mermaid)

A visual map of how Copilot Studio agents are **built**, how they **run**, and how an enterprise **wraps** identity, governance, ALM, and cost around them. Nine schematic plates, read as a set.

> Compiled 2026-07-23 · Source: Microsoft Learn + local knowledge base · Scope: classic + generative orchestration.
> Renders automatically on GitHub and in VS Code Markdown preview with a Mermaid extension. Companion to the styled page `copilot-studio-architecture.html` and to `knowledge-base/reference/architecture-diagrams.md`.

**Three ways to read the plates**

| Phase | Meaning | Plates |
|-------|---------|--------|
| **Build time** | Author components as YAML, version in Git, promote through environments | 02, 06 |
| **Run time** | The orchestrator plans each turn — grounds, calls tools, delegates, moderates, summarizes | 03, 04 |
| **Enterprise wrap** | Identity, DLP, managed environments, monitoring, multi-agent, channels, cost | 01, 05, 07–09 |

---

## Plate 01 — The enterprise platform stack

**Read this as layers.** Everything rests on Entra ID (identity), Dataverse (storage), and Power Platform (governance). Governance isn't a layer — it wraps all of them.

```mermaid
flowchart TB
  subgraph L6["SURFACES / CHANNELS"]
    direction LR
    Teams["Teams and M365 Copilot"]
    Web["Web / Direct Line"]
    Wapp["WhatsApp"]
    Tel["Telephony / voice"]
  end
  subgraph L5["COPILOT STUDIO AGENT"]
    direction LR
    Instr["Instructions"]
    Know["Knowledge"]
    Tools["Tools and Skills"]
    Topics["Topics"]
    Trig["Triggers"]
    Sub["Child / Connected agents"]
  end
  subgraph L4["ORCHESTRATION RUNTIME"]
    direction LR
    Planner["Generative orchestrator"]
    Classic["Classic orchestrator"]
    Mod["Content moderation"]
  end
  subgraph L3["MODELS"]
    direction LR
    GPT["GPT / reasoning models"]
    STS["Real-time speech-to-speech"]
  end
  subgraph L2["INTEGRATION"]
    direction LR
    Conn["Connectors"]
    Flows["Power Automate / Agent flows"]
    MCP["MCP servers"]
    REST["REST / HTTP"]
    A2A["A2A / Foundry / Fabric / SDK"]
  end
  subgraph L1["FOUNDATION — MICROSOFT CLOUD"]
    direction LR
    Entra["Microsoft Entra ID"]
    DV["Dataverse"]
    PP["Power Platform"]
    Graph["Microsoft 365 / Graph"]
  end
  L6 --> L5 --> L4 --> L3
  L5 --> L2
  L4 --> L2
  L2 --> L1
  L5 --> L1
  Gov["GOVERNANCE and SECURITY — DLP, Managed Environments, Purview, Sentinel, CMK"]
  Gov -. "wraps all layers" .-> L5
  Gov -.-> L2
  Gov -.-> L1
```

---

## Plate 02 — Anatomy of a single agent

**Read this as a parts list.** An agent is these components. Under generative orchestration the planner routes across them using their **names and descriptions** — which is why descriptions are the real source code.

```mermaid
flowchart LR
  Agent(("AGENT"))
  Agent --> I["Instructions — identity, scope, rules"]
  Agent --> K["Knowledge — RAG grounding"]
  Agent --> T["Tools and Skills — actions"]
  Agent --> TP["Topics — deterministic dialog"]
  Agent --> TR["Triggers — when it activates"]
  Agent --> M["Model — reasoning LLM"]
  Agent --> S["Child / Connected agents"]
  Agent --> MEM["Memory — per-user, preview"]
  K --> K2["SharePoint / OneDrive"]
  K --> K3["Dataverse tables"]
  K --> K4["Graph connectors"]
  K --> K1["Public web / Bing"]
  T --> T1["Power Automate / Agent flow"]
  T --> T2["Prompt tool"]
  T --> T3["Connector / Custom connector"]
  T --> T4["REST API"]
  T --> T5["MCP server"]
  T --> T6["Computer Use / CUA"]
```

---

## Plate 03 — Runtime flow (generative orchestration)

**Read this as one user turn.** The planner reads instructions + descriptions, builds a plan, grounds and acts in parallel, moderates, then summarizes a cited answer. The activity map exposes this path when testing.

```mermaid
sequenceDiagram
  actor U as User
  participant CH as Channel
  participant AU as Entra ID
  participant OR as Orchestrator
  participant LLM as Model
  participant KN as Knowledge
  participant TL as Tools
  participant SA as Connected agents
  participant MD as Moderation
  U->>CH: Message or event trigger
  CH->>AU: Authenticate (SSO)
  AU-->>CH: Identity + token
  CH->>OR: Intent + conversation history
  OR->>LLM: Build plan from instructions + descriptions
  LLM-->>OR: Plan (knowledge / tools / topics / agents)
  par Ground and act
    OR->>KN: Retrieve (trimmed to user access)
    KN-->>OR: Passages + citations
  and
    OR->>TL: Call tools with AI-filled inputs
    TL-->>OR: Results
  and
    OR->>SA: Delegate task
    SA-->>OR: Findings only
  end
  OR->>MD: Moderate
  OR->>LLM: Summarize grounded response
  LLM-->>OR: Answer + citations
  OR-->>CH: Response
  CH-->>U: Answer
```

> **Cost note:** the planner reasons every turn, so generative orchestration costs more credits than a classic topic-trigger match. Use classic for high-volume, predictable intents.

---

## Plate 04 — Knowledge & RAG pipeline

**Read this as grounding.** Sources are indexed, retrieved (description-filtered past 25 sources), **security-trimmed to the asking user**, then generated. No citation + ungrounded off = the answer is withheld.

```mermaid
flowchart LR
  S1["Public websites"] --> IDX
  S2["SharePoint / OneDrive"] --> IDX
  S3["Dataverse tables"] --> IDX
  S4["Graph connectors"] --> IDX
  S5["Uploaded files"] --> IDX
  IDX["Index / semantic index"] --> RET
  Q["User question"] --> RET["Retrieve (filtered if over 25 sources)"]
  RET --> TRIM["Security trimming — user sees only permitted data"]
  TRIM --> GEN["Generate grounded answer"]
  GEN --> CITE{"Has citation?"}
  CITE -- yes --> OUT["Answer + citations"]
  CITE -- "no, ungrounded off" --> FB["Withheld — fallback topic fires"]
```

---

## Plate 05 — Multi-agent topology (hub & spoke)

**Read this as delegation.** A lean orchestrator routes to independently-owned domain agents. Only the orchestrator talks to the user; sub-agents return findings. Split past ~30–40 combined choices.

```mermaid
flowchart TB
  U["Employee / customer"] --> OA["Orchestrator agent — routing only"]
  OA -->|"HR intent"| HR["HR Agent"]
  OA -->|"IT intent"| IT["IT Support Agent"]
  OA -->|"Finance intent"| FIN["Finance Agent"]
  OA -->|"low confidence"| HUM["Human escalation"]
  HR --> HRK["SharePoint HR policies"]
  IT --> ITK["Dataverse + ServiceNow"]
  FIN --> FBK["Fabric Data agent"]
  OA -. "A2A / SDK / Foundry" .-> EXT["External agents"]
```

> **Trade-off:** each hop adds latency and credits. Keep the parent a router, not an executor — don't duplicate sub-agent capability in the parent.

---

## Plate 06 — ALM & the code-first dev loop

**Read this as the pipeline.** Clone to VS Code, edit YAML, validate, commit. Apply to DEV to test; gate promotion on test sets via solutions + pipelines. **Apply is not Publish.**

```mermaid
flowchart LR
  subgraph Local["DEVELOPER — VS CODE"]
    Clone["Clone agent (.mcs.yml)"] --> Edit["Edit YAML (AI-assisted)"]
    Edit --> Val["Validate — schema, IDs, Power Fx"]
    Val --> Git["Git commit / PR"]
  end
  subgraph Dev["DEV"]
    Apply["Apply → test pane"] --> Eval["Evaluate — test sets"]
  end
  subgraph Test["TEST"]
    Kit["Agent Kit — batch tests + rubrics"]
  end
  subgraph Prod["PROD"]
    Pub["Publish → channels"]
    Mon["Monitor / Analytics"]
  end
  Git --> Apply
  Eval -->|"solution"| Pipe["Power Platform Pipelines"]
  Pipe --> Kit
  Kit -->|"pass"| Pub
  Mon -->|"real convos become test cases"| Eval
```

> **Per-environment:** DLP policies, connection references, and environment variables rebind on each hop — solutions and pipelines handle this.

---

## Plate 07 — Governance & security layers

**Read this as the wrap.** Four control planes surround every agent: identity, data controls, environment governance, and audit/monitoring.

```mermaid
flowchart TB
  Agent(("AGENT"))
  Agent --- ID["IDENTITY — Entra ID auth, cert, tool creds (end-user vs maker)"]
  Agent --- DC["DATA CONTROLS — DLP policies, endpoint filtering, sensitivity labels, CMK"]
  Agent --- EG["ENVIRONMENT — Managed Environments, routing, security scan"]
  Agent --- OB["AUDIT — Purview logs, Sentinel alerts, App Insights, CoE / Agent Inventory"]
```

---

## Plate 08 — Channels & surfaces

**Read this as reach.** One published agent, many surfaces — each renders differently (Teams caps cards/citations; telephony is voice-only).

```mermaid
flowchart LR
  Agent(("PUBLISHED AGENT"))
  Agent --> Teams["Microsoft Teams"]
  Agent --> M365["Microsoft 365 Copilot"]
  Agent --> Webc["Custom website / Web Chat"]
  Agent --> DL["Direct Line — custom apps"]
  Agent --> SP["SharePoint"]
  Agent --> WA["WhatsApp"]
  Agent --> Tele["Telephony — ACS / D365 Contact Center"]
  Agent --> ABS["Azure Bot Service — Slack, Telegram, Email"]
```

---

## Plate 09 — Cost model (Copilot Credits)

**Read this as the meter.** Each turn can stack meters; all draw on one tenant pool. Cross 125% of prepaid capacity and agents are disabled — unless the environment is pay-as-you-go.

```mermaid
flowchart LR
  Turn["One agent turn"] --> C1["Classic answer = 1"]
  Turn --> C2["Generative answer = 2"]
  Turn --> C3["Agent action / CUA step = 5"]
  Turn --> C4["Tenant graph grounding = 10"]
  Turn --> C5["Agent flow = 13 per 100 actions"]
  Turn --> C6["AI tools = 1 / 15 / 100"]
  C1 --> Pool["Tenant credit pool"]
  C2 --> Pool
  C3 --> Pool
  C4 --> Pool
  C5 --> Pool
  C6 --> Pool
  Pool --> OVR{"Over 125% of prepaid?"}
  OVR -- yes --> Block["Agents disabled — unless pay-as-you-go"]
  OVR -- no --> Run["Runs normally"]
```

---

### How to read the whole set together
1. **Build time** (plates 02, 06): author components as YAML, version in Git, promote via solutions/pipelines.
2. **Run time** (plates 03, 04): the orchestrator plans per turn, grounds on knowledge, calls tools, delegates, moderates, summarizes.
3. **Enterprise wrap** (plates 01, 05, 07–09): identity, DLP, managed environments, monitoring, multi-agent topology, channels, and cost governance surround everything.

Deep-dive references live in [knowledge-base/](knowledge-base/): fundamentals, generative orchestration, knowledge, governance/DLP, licensing/credits, ALM, and multi-agent patterns.
