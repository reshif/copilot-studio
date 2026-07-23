# Reference — Copilot Studio Architecture (Mermaid diagrams)

Visual model of Copilot Studio for **enterprise** use. Each diagram has a short "read this as…" note. (Diagrams render on GitHub, in an Artifact, or in VS Code with a Mermaid preview extension.)

---

## 1. The enterprise platform stack (layers)

How the pieces sit on top of Microsoft's cloud. Everything an agent does is built on **Entra ID (identity)**, **Dataverse (storage)**, and **Power Platform (governance/ALM)**.

```mermaid
flowchart TB
    subgraph L6["Surfaces / Channels"]
        direction LR
        Teams["Teams & M365 Copilot"]
        Web["Web / Direct Line"]
        Wapp["WhatsApp"]
        Tel["Telephony (voice)"]
        Other["Facebook, Slack, Email…"]
    end

    subgraph L5["Copilot Studio — Agent"]
        direction LR
        Instr["Instructions"]
        Know["Knowledge"]
        Tools["Tools & Skills"]
        Topics["Topics"]
        Trig["Triggers"]
        Sub["Child / Connected agents"]
    end

    subgraph L4["Orchestration runtime"]
        Planner["Generative orchestrator (LLM planner)"]
        Classic["Classic orchestrator (trigger-phrase)"]
        Mod["Content moderation"]
    end

    subgraph L3["Models"]
        GPT["GPT-family / reasoning models"]
        STS["Real-time speech-to-speech"]
    end

    subgraph L2["Integration"]
        direction LR
        Conn["Power Platform connectors"]
        Flows["Power Automate / Agent flows"]
        MCP["MCP servers"]
        REST["REST / HTTP"]
        A2A["A2A / Foundry / Fabric / Agents SDK"]
    end

    subgraph L1["Foundation — Microsoft Cloud"]
        direction LR
        Entra["Microsoft Entra ID"]
        DV["Dataverse"]
        PP["Power Platform (environments, solutions)"]
        Graph["Microsoft 365 / Graph"]
    end

    L6 --> L5 --> L4 --> L3
    L5 --> L2
    L4 --> L2
    L2 --> L1
    L5 --> L1

    Gov["Governance & Security<br/>DLP · Managed Environments · Purview · Sentinel · CMK"]
    Gov -. "wraps every layer" .-> L5
    Gov -.-> L2
    Gov -.-> L1
```

---

## 2. Anatomy of a single agent

What an "agent" actually is. The orchestrator routes across these components using their **names + descriptions**.

```mermaid
flowchart LR
    Agent(("Agent"))
    Agent --> I["Instructions<br/><i>identity, scope, rules</i>"]
    Agent --> K["Knowledge<br/><i>RAG grounding sources</i>"]
    Agent --> T["Tools & Skills<br/><i>actions: flows, connectors, REST, MCP, CUA</i>"]
    Agent --> TP["Topics<br/><i>deterministic dialog (AdaptiveDialog)</i>"]
    Agent --> TR["Triggers<br/><i>when it activates</i>"]
    Agent --> M["Model<br/><i>reasoning LLM</i>"]
    Agent --> S["Child / Connected agents<br/><i>delegation</i>"]
    Agent --> MEM["Memory<br/><i>per-user, preview</i>"]

    K --> K1["Public website / Bing"]
    K --> K2["SharePoint / OneDrive"]
    K --> K3["Dataverse tables"]
    K --> K4["Graph connectors"]
    K --> K5["Uploaded files"]

    T --> T1["Power Automate / Agent flow"]
    T --> T2["Prompt tool"]
    T --> T3["Connector / Custom connector"]
    T --> T4["REST API"]
    T --> T5["MCP server"]
    T --> T6["Computer Use (CUA)"]
```

---

## 3. Runtime request flow (generative orchestration)

The critical path for **one user turn**. This is why "descriptions are the real source code" — the planner selects components from them.

```mermaid
sequenceDiagram
    actor U as User
    participant CH as Channel
    participant AU as Entra ID (auth/SSO)
    participant OR as Orchestrator (planner)
    participant LLM as Model
    participant KN as Knowledge (RAG)
    participant TL as Tools / Flows
    participant SA as Connected agents
    participant MD as Moderation

    U->>CH: Message or event trigger
    CH->>AU: Authenticate
    AU-->>CH: Identity + token
    CH->>OR: Intent + conversation history
    OR->>LLM: Build plan from instructions + descriptions
    LLM-->>OR: Plan (which knowledge/tools/topics/agents)
    par Grounding & actions
        OR->>KN: Retrieve (security-trimmed to user)
        KN-->>OR: Passages + citations
    and
        OR->>TL: Call tool(s) with AI-filled inputs
        TL-->>OR: Results
    and
        OR->>SA: Delegate task (optional)
        SA-->>OR: Findings only
    end
    OR->>MD: Moderate
    OR->>LLM: Summarize grounded response
    LLM-->>OR: Answer + in-text citations
    OR-->>CH: Response
    CH-->>U: Answer (activity map shows the path)
```

---

## 4. Knowledge / RAG pipeline

How grounding works, and where enterprise controls (auth trimming, semantic index, moderation) sit.

```mermaid
flowchart LR
    subgraph Sources
        S1["Public websites"]
        S2["SharePoint / OneDrive"]
        S3["Dataverse tables"]
        S4["Graph connectors"]
        S5["Uploaded files"]
    end
    S1 & S2 & S3 & S4 & S5 --> IDX["Index / semantic index<br/>(tenant graph grounding)"]
    Q["User question"] --> RET["Retrieve<br/>(description-filtered if >25 sources)"]
    IDX --> RET
    RET --> TRIM["Security trimming<br/>(user only sees permitted data)"]
    TRIM --> GEN["Generate grounded answer<br/>SearchAndSummarizeContent"]
    GEN --> CITE{"Has citation?"}
    CITE -- yes --> OUT["Answer + citations"]
    CITE -- "no & ungrounded OFF" --> FB["Blocked → fallback topic"]
    GEN --> MODP["Moderation level"]
```

---

## 5. Multi-agent enterprise topology (hub-and-spoke)

The pattern most enterprises land on: a lean **orchestrator** routing to independently-owned **domain agents**. Keep the parent a router, not an executor.

```mermaid
flowchart TB
    U["Employee / customer"] --> OA["Orchestrator agent<br/><i>routing only, lean instructions</i>"]

    OA -->|"HR intent"| HR["HR Agent"]
    OA -->|"IT intent"| IT["IT Support Agent"]
    OA -->|"Finance intent"| FIN["Finance Agent"]
    OA -->|"low confidence"| HUM["Human escalation"]

    HR --> HRK["SharePoint HR policies"]
    IT --> ITK["Dataverse + ServiceNow"]
    FIN --> FBK["Fabric Data agent"]

    OA -.A2A / SDK / Foundry.-> EXT["External agents<br/>(other frameworks)"]

    classDef owned fill:#eef,stroke:#88a;
    class HR,IT,FIN owned
```

> Rule of thumb: split into connected agents past **~30–40 combined choices** or when teams own domains separately. Each hop adds latency + credits. Single-response principle: **only the orchestrator talks to the user**.

---

## 6. Enterprise ALM + code-first dev loop

How agents move from a developer's machine to production **safely and repeatably** — the framework we're building toward.

```mermaid
flowchart LR
    subgraph Local["Developer (VS Code)"]
        Clone["Clone agent<br/>(.mcs.yml)"] --> Edit["Edit YAML<br/>(AI-assisted)"]
        Edit --> Val["Validate<br/>(schema, IDs, Power Fx)"]
        Val --> Git["Git commit / PR"]
    end

    subgraph Dev["DEV environment"]
        Apply["Apply → test pane"] --> Eval["Evaluate (test sets)"]
    end

    subgraph Test["TEST environment"]
        Kit["Copilot Agent Kit<br/>batch tests + rubrics"]
    end

    subgraph Prod["PROD environment"]
        Pub["Publish → channels"]
        Mon["Monitor / Analytics"]
    end

    Git --> Apply
    Eval -->|"solution"| Pipe["Power Platform Pipelines<br/>(gated on tests)"]
    Pipe --> Kit
    Kit -->|"pass"| Pipe2["Promote"]
    Pipe2 --> Pub
    Mon -->|"real convos → new test cases"| Eval

    DLP["DLP policies per environment"] -.-> Dev
    DLP -.-> Test
    DLP -.-> Prod
```

> **Apply ≠ Publish.** Apply updates the live definition for testing; Publish exposes it on channels. Connection references & environment variables rebind per environment (solutions/pipelines handle this).

---

## 7. Governance & security layers

The cross-cutting controls an enterprise wraps around every agent.

```mermaid
flowchart TB
    subgraph Identity
        SSO["Entra ID auth<br/>(Authenticate with Microsoft / manual / cert)"]
        Cred["Tool creds: end-user vs maker"]
    end
    subgraph DataControls["Data controls"]
        DLP["DLP data policies<br/>(Business / Non-business / Blocked)"]
        EPF["Endpoint filtering"]
        Sens["Sensitivity labels (Purview)"]
        CMK["Customer-managed keys"]
    end
    subgraph EnvGov["Environment governance"]
        ME["Managed Environments"]
        Route["Environment routing"]
        Scan["Automatic security scan"]
    end
    subgraph Observe["Audit & monitoring"]
        Purview["Purview audit logs"]
        Sentinel["Sentinel alerts"]
        AppI["Application Insights"]
        CoE["CoE Starter Kit / Agent Inventory"]
    end

    Agent(("Agent")) --- Identity
    Agent --- DataControls
    Agent --- EnvGov
    Agent --- Observe
```

---

## 8. Channels & surfaces (where users meet the agent)

One published agent, many surfaces — each with different rendering (see limits per channel).

```mermaid
flowchart LR
    Agent(("Published agent")) --> Teams["Microsoft Teams"]
    Agent --> M365["Microsoft 365 Copilot"]
    Agent --> Web["Custom website / Web Chat"]
    Agent --> DL["Direct Line (custom apps)"]
    Agent --> SP["SharePoint"]
    Agent --> WA["WhatsApp"]
    Agent --> Tele["Telephony (ACS / D365 Contact Center)"]
    Agent --> ABS["Azure Bot Service: Slack, Telegram, Email…"]
```

---

## 9. Cost model (Copilot Credits, simplified)

Where credits are consumed — shapes design (classic vs generative, tools, autonomous frequency).

```mermaid
flowchart LR
    Turn["One agent turn"] --> C1["Classic answer = 1"]
    Turn --> C2["Generative answer = 2"]
    Turn --> C3["Agent action / CUA step = 5"]
    Turn --> C4["Tenant graph grounding = 10"]
    Turn --> C5["Agent flow = 13 per 100 actions"]
    Turn --> C6["AI tools: 1 / 15 / 100 (basic/std/premium)"]
    C1 & C2 & C3 & C4 & C5 & C6 --> Pool["Tenant credit pool"]
    Pool --> OVR{">125% of prepaid?"}
    OVR -- yes --> Block["Agents disabled (unless pay-as-you-go)"]
    OVR -- no --> Run["Runs normally"]
```

---

## How to read the whole thing together
1. **Build time** (diagrams 2, 6): you author components as YAML, version in Git, promote via solutions/pipelines.
2. **Run time** (diagrams 3, 4): the orchestrator plans per turn, grounds on knowledge, calls tools, delegates to agents, moderates, and summarizes.
3. **Enterprise wrap** (diagrams 1, 5, 7, 8, 9): identity, DLP, managed environments, monitoring, multi-agent topology, channels, and cost governance surround everything.

Cross-references: [../01-copilot-studio-fundamentals.md](../01-copilot-studio-fundamentals.md) · [generative-orchestration.md](generative-orchestration.md) · [knowledge.md](knowledge.md) · [governance-security-dlp.md](governance-security-dlp.md) · [licensing-and-credits.md](licensing-and-credits.md) · [../10-alm-publishing-testing.md](../10-alm-publishing-testing.md) · [../guidance/multi-agent-patterns.md](../guidance/multi-agent-patterns.md).
