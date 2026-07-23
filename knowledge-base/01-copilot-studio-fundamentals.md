# 01 — Copilot Studio Fundamentals

## What is Microsoft Copilot Studio?

Copilot Studio is Microsoft's **agent-building platform**. It's an end-to-end conversational and autonomous AI platform that lets you design, test, and publish AI **agents** using natural language and/or a graphical interface, and now also **code-first via YAML** in VS Code.

An agent built in Copilot Studio can:

- **Answer questions** using public or enterprise knowledge (RAG / generative answers).
- **Execute actions** through integrated tools (connectors, flows, APIs).
- **Handle** simple to complex **conversational and autonomous flows**.
- **Deploy to channels** (Teams, web, Microsoft 365 Copilot, WhatsApp, etc.).
- Be built as **workflow-based agents** *and* **fully AI-driven agents** with a built-in orchestrator.
- Be **autonomous** (triggered by events, acts on its own) *or* **conversational** (responds to users).

## Where it sits in the Microsoft stack

- Built on **Power Platform** and **Dataverse** (agents live inside a Dataverse **environment**; ALM uses **solutions**).
- Formerly **Power Virtual Agents** (that's why old community/docs URLs say `power-virtual-agents`).
- Integrates with **Power Automate** (flows as tools), **Power Fx** (low-code expression language), **Microsoft 365 / Graph** (knowledge + connectors), **Microsoft Foundry**, **Microsoft Fabric**, and the **Microsoft 365 Agents SDK** (for connected external agents).
- Adjacent developer tooling: the **Copilot Studio extension for VS Code** (code-first authoring — the focus of this KB) and the **Microsoft 365 Agents Toolkit / Agents SDK** (pro-code agents outside Copilot Studio that can be *connected* to Copilot Studio agents).

## The core anatomy of an agent

Every agent — regardless of experience or authoring surface — is organized around these components:

| Component | What it is |
|-----------|-----------|
| **Instructions** | The agent's identity, tone, scope, behavior (the system prompt). |
| **Knowledge** | Connected data sources that ground answers: public websites, SharePoint, Graph connectors, uploaded files, Dataverse, custom APIs; plus **memory** and organizational data via Microsoft IQ. |
| **Tools & skills** | Actions the agent can take: connector actions, Power Automate flows, prompts, custom connectors, REST APIs, MCP connectors. Skills = reusable structured behaviors. |
| **Model** | The LLM that powers reasoning (e.g., GPT-family models; GPT-5 Chat is GA in Copilot Studio as of late Nov 2025). |
| **Topics** | Scripted conversational flows / dialogs (classic strength; still available). A topic is technically an `AdaptiveDialog`. |
| **Triggers** | Define *when* topics or actions activate — conversation start, recognized intent, activity, schedule, event, or external/workflow trigger. |
| **Connected / child agents** | Delegate specialized tasks to other agents (multi-agent orchestration). |

## Agent types (two useful axes)

1. **Conversational vs Autonomous**
   - *Conversational*: user talks to it; it responds (classic chatbot + generative answers).
   - *Autonomous*: triggered by events/data changes; reasons and acts with tools, potentially without a user turn.

2. **Workflow-based vs AI-driven (orchestrated)**
   - *Workflow-based*: you author explicit topics/flows and branching logic (deterministic).
   - *AI-driven*: a built-in **orchestrator** decides which tools/topics/knowledge/agents to use based on natural-language **descriptions** (generative orchestration). This is the direction of the **new experience** (see [02](02-agent-experience-new-vs-classic.md)).

Most real agents are a **blend**: generative orchestration for flexibility, with a few authored topics for deterministic, compliance-sensitive flows.

## Key vocabulary you'll see constantly

- **Maker** — anyone building an agent (low-code persona).
- **Environment** — a Dataverse container (dev / test / prod / sandbox / default). Agents belong to one.
- **Solution** — the ALM packaging unit used to move agents between environments.
- **Orchestration** — the runtime deciding how to answer: which knowledge, tool, topic, or connected agent to use.
- **Generative answers** — RAG over your knowledge sources (the `SearchAndSummarizeContent` node) or general-model answers (`AnswerQuestionWithAI`).
- **Channel** — where the agent is published/surfaced (Teams, custom website, M365 Copilot, WhatsApp, DirectLine, etc.).
- **`.mcs.yml`** — the Copilot Studio YAML authoring file format (MCS = Microsoft Copilot Studio).

## What's new / notable (2025–2026)

- **New agent experience** (production-ready preview): natural-language-first, single-surface, enhanced orchestration. See [02](02-agent-experience-new-vs-classic.md).
- **Multi-agent orchestration** GA/preview: connect child agents and external agents (Foundry, Fabric, M365 Agents SDK, A2A). See [09](09-multi-agent-orchestration.md).
- **Copilot Studio extension for VS Code → GA on 2026-01-14**: code-first, clone/edit/sync agent definitions. See [03](03-vscode-extension.md).
- **New publishing channels**: WhatsApp (from July 2025).
- **Model updates**: GPT-5 Chat GA in Copilot Studio (US + EU) from 2025-11-24.

See [12-glossary-and-sources.md](12-glossary-and-sources.md) for sources.
