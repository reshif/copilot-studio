# 12 — Glossary & Sources

## Glossary

- **Agent** — an AI assistant built in Copilot Studio (conversational and/or autonomous), defined by instructions, knowledge, tools, model, topics, triggers, and optionally connected/child agents.
- **AdaptiveDialog** — the `kind` of a topic; a dialog with a `beginDialog` trigger and ordered `actions` (nodes).
- **ALM** — Application Lifecycle Management; here, moving agents dev → test → prod via solutions/pipelines.
- **Apply** — VS Code extension op that pushes local changes live to an environment's agent definition (not a publish).
- **Autonomous agent** — triggered by events; reasons/acts with tools, potentially without a user turn.
- **Channel** — where a published agent runs (Teams, web/DirectLine, M365 Copilot, WhatsApp…).
- **Child agent** — a lightweight agent nested inside a main agent (`kind: AgentDialog`, under `agents/`).
- **Connected agent** — a separate agent linked to a main agent (same-env Copilot Studio, or external via A2A/Foundry/Fabric/M365 Agents SDK).
- **Connection reference** — environment-specific binding for a connector's auth (`connectionreferences.mcs.yml`).
- **Dataverse** — the data platform under Power Platform where agents/environments live.
- **Environment** — a Dataverse container (dev/test/prod/sandbox/default).
- **Generative answers** — RAG over knowledge (`SearchAndSummarizeContent`) or general-model answers (`AnswerQuestionWithAI`).
- **Generative orchestration** — the runtime choosing tools/topics/knowledge/agents by their natural-language descriptions (`grounding.GenerativeActionsEnabled`).
- **Get / Preview** — VS Code extension sync ops (cloud → local): apply remote changes / view remote changes.
- **Knowledge source** — grounding data (`KnowledgeSourceConfiguration`): public site, SharePoint, Graph connector, files, custom API.
- **Maker** — anyone building an agent.
- **`.mcs.yml`** — Microsoft Copilot Studio YAML authoring file format.
- **Microsoft 365 Agents SDK / Toolkit** — pro-code path for building agents (can be *connected* to Copilot Studio agents).
- **Model** — the LLM powering reasoning (e.g., GPT-5 Chat, GA in Copilot Studio Nov 2025).
- **New experience** — natural-language-first authoring with enhanced orchestration (production-ready preview); no conversion to/from classic.
- **Orchestration** — runtime decision layer for how to answer a turn.
- **Power Fx** — Excel-like low-code expression language; expressions prefixed with `=`.
- **Publish** — makes the agent available to end users on its channels (distinct from Apply).
- **Reattach** — `Copilot Studio: Reattach Agent`, connects a local folder to a cloud agent/environment.
- **Solution** — Power Platform packaging unit for ALM.
- **Tool** — an action the agent can take (connector, flow, prompt, REST, MCP, custom connector); `kind: TaskDialog` under `actions/`.
- **Topic** — a scripted conversational flow (`AdaptiveDialog`) under `topics/`.
- **Trigger** — what activates a topic/action (`OnConversationStart`, `OnRecognizedIntent`, `OnActivity`, `OnUnknownIntent`, external/workflow…).
- **Variable scopes** — `System.`, `Global.`, `Topic.`, `Environment.`.

## Sources (accessed 2026-07-23)

**Microsoft Learn — Copilot Studio (official docs)**
- Official Copilot Studio documentation — https://learn.microsoft.com/en-us/microsoft-copilot-studio/
- VS Code extension overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-overview
- VS Code extension install/configure — https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-install-configure
- VS Code extension clone agent — https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-clone-agent
- VS Code extension edit components — https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-edit-agent-components
- VS Code extension synchronization — https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-synchronization
- Agents overview (new experience) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/overview
- Classic vs new agent experience — https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/classic-vs-new
- Use the code editor in topics (YAML) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/topics-code-editor
- Create expressions using Power Fx — https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-power-fx
- Power Fx formula reference (Copilot Studio) — https://learn.microsoft.com/en-us/power-platform/power-fx/formula-reference-copilot-studio
- Work with variables — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-variables
- Add other agents overview (multi-agent) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-add-other-agents
- Multi-agent orchestration patterns — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns
- What's new in Copilot Studio — https://learn.microsoft.com/en-us/microsoft-copilot-studio/whats-new

**Microsoft Learn — deep-dive pages (added in expansion 2)**
- Write agent instructions — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-instructions
- Configure high-quality instructions (generative mode guidance) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/generative-mode-guidance
- Create and edit topics (node types) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-create-edit-topics
- Use entities and slot filling — https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-entities-slot-filling
- Variables overview (system variables) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-variables-about
- Orchestrate with generative AI — https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-generative-actions
- Add tools to custom agents — https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-tools-custom-agent
- Knowledge sources summary — https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio
- Set topic triggers — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-triggers
- Make HTTP requests — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-http-node
- Publish and deploy (channels) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/publication-fundamentals-publish-channels
- Add user authentication to topics — https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-end-user-authentication
- Quotas and limits — https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-quotas
- Prompts (prompt tool / AI Builder) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/nlu-prompt-node
- Agent flows overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/flows-overview
- Ask with Adaptive Cards — https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-ask-with-adaptive-card
- Computer use (CUA) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/computer-use
- Analytics overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-overview
- Choose evaluation methods — https://learn.microsoft.com/en-us/microsoft-copilot-studio/analytics-agent-evaluation-overview
- Billing rates and management (Copilot Credits) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management
- Security and governance — https://learn.microsoft.com/en-us/microsoft-copilot-studio/security-and-governance
- Configure data policies for agents (DLP) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/admin-data-loss-prevention
- Multi-agent orchestration patterns — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns
- Extend Microsoft 365 Copilot with agents — https://learn.microsoft.com/en-us/microsoft-copilot-studio/microsoft-365-copilot-extend-with-agents
- Memory (preview, new experience) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/memory-overview
- Create a custom agent from a template — https://learn.microsoft.com/en-us/microsoft-copilot-studio/template-fundamentals
- Agent Library (Copilot Agent Kit) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-library-overview
- Guidance hub — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/

**Microsoft Learn — niche deep-dives (added in expansion 3)**
- Interactive voice response overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/voice-overview
- Configure basic voice agents — https://learn.microsoft.com/en-us/microsoft-copilot-studio/voice-configuration
- Extend your agent with MCP — https://learn.microsoft.com/en-us/microsoft-copilot-studio/agent-extend-action-mcp
- Configure skills for agents — https://learn.microsoft.com/en-us/microsoft-copilot-studio/configuration-add-skills
- Connect over Agent2Agent (A2A) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-agent-agent-to-agent
- Add Dataverse tables as knowledge — https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-dataverse
- Copilot Agent Kit overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-overview
- Power CAT Copilot Studio Kit (GitHub) — https://github.com/microsoft/Power-CAT-Copilot-Studio-Kit
- CopilotStudioSamples (A2A/extensibility) — https://github.com/microsoft/CopilotStudioSamples

**Microsoft Learn — the guidance library (best practices; added in expansion 4)**
- Guidance hub (index of all best-practice articles) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/
- Apply generative orchestration capabilities — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/generative-orchestration
- FAQ for generative orchestration — https://learn.microsoft.com/en-us/microsoft-copilot-studio/faqs-generative-orchestration
- Design autonomous agent capabilities — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/autonomous-agents
- Use agent tools to extend, automate, enhance — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-tools
- Design effective agents (structured design framework) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-design-canvas-overview
- Topic authoring best practices — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/topic-authoring-best-practices
- Design effective trigger phrases — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/trigger-phrases-best-practices
- Slot-filling best practices — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/slot-filling-best-practices
- Enhance AI responses with RAG — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/retrieval-augmented-generation
- Choose the right platform — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/technical-readiness
- Improve conversational agent performance — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/conversational-agents-performance-improvement
- Optimize prompts overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/optimize-prompts-overview
- Plan for throughput and rate limits — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/plan-agent-throughput-rate-limits
- Agent flow express mode (preview) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/agent-flow-express-mode
- Multi-agent orchestration patterns — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/multi-agent-patterns
- Copilot Agent Kit overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/kit-overview

**Microsoft Learn — guidance library, Plan/Manage/Improve branches (expansion 5)**
- Define project objectives and success criteria — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/project-best-practices
- Measure ROI and business value of AI agents — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-business-value-overview
- Define your solution architecture — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/architecture-overview
- Apply responsible AI principles — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/responsible-ai
- Introduction to conversational experiences — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/cux-overview
- Principles of conversational experience design — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/cux-principles
- Design effective language understanding — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/language-understanding
- Connect to custom knowledge sources (OnKnowledgeRequested) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/custom-knowledge-sources
- Manage your projects (security & governance overview) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/sec-gov-intro
- Implement a zoned governance strategy — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/sec-gov-phase2
- Design a testing strategy — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/sec-gov-phase4
- Establish an ALM strategy — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/alm
- Measure agent engagement — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/measuring-engagement
- Measure agent outcomes — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/measuring-outcomes
- Deflection overview — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/deflection-overview
- Evaluation-driven triage and remediation — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/evaluation-triage-overview
- Power Platform Well-Architected — https://learn.microsoft.com/en-us/power-platform/well-architected
- Architecting agent solutions — https://learn.microsoft.com/en-us/agents/architecture/

**Microsoft Learn — final branches (expansion 6)**
- Measure the impact of your agents (4 value drivers, Agent Assisted Hours) — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-business-value-measure-impact
- Agent metrics reference library — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-business-value-metrics-reference
- Use case blueprints for measuring agent value — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/agent-business-value-use-case-blueprints
- Measure and improve performance with KPIs and analytics — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/analytics
- Agentic AI adoption maturity model — https://learn.microsoft.com/en-us/agents/adoption-maturity-model/
- Plan and create a conversational agent performance test — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/conversational-agents-performance-testing
- Develop a custom analytics strategy — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/custom-analytics-strategy
- Plan and design integration strategies — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/integrations
- Secure your Copilot Studio projects — https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/sec-gov-phase3
- Copilot Studio agents report (Viva Insights, AAH calculator) — https://learn.microsoft.com/en-us/viva/insights/advanced/analyst/templates/copilot-studio-agents

**Microsoft blogs**
- Copilot Studio extension for VS Code is GA (M365 Dev Blog, 2026-01) — https://devblogs.microsoft.com/microsoft365dev/copilot-studio-extension-for-visual-studio-code-is-now-generally-available/
- Multi-agent orchestration & more (Build 2025 announcements) — https://www.microsoft.com/en-us/microsoft-copilot/blog/copilot-studio/multi-agent-orchestration-maker-controls-and-more-microsoft-copilot-studio-announcements-at-microsoft-build-2025/
- Product page — https://www.microsoft.com/en-us/microsoft-365-copilot/microsoft-copilot-studio

**Code-first framework**
- microsoft/skills-for-copilot-studio (repo) — https://github.com/microsoft/skills-for-copilot-studio
- skills-for-copilot-studio — Skills reference (DeepWiki) — https://deepwiki.com/microsoft/skills-for-copilot-studio/4-skills-reference
- skills-for-copilot-studio — YAML authoring model (DeepWiki) — https://deepwiki.com/microsoft/skills-for-copilot-studio/2.2-yaml-authoring-model
- Skills for Copilot Studio blog — https://microsoft.github.io/mcscatblog/posts/skills-for-copilot-studio/
- Extension GitHub (issues/roadmap) — https://github.com/microsoft/vscode-copilotstudio/issues
- VS Code Marketplace — https://marketplace.visualstudio.com/items?itemName=ms-CopilotStudio.vscode-copilotstudio

**Community deep-dives (secondary, useful but verify against Learn)**
- Holger Imbery — Power Fx in Copilot Studio; VS Code extension notes — https://holgerimbery.blog/
- Visual Studio Magazine — Extension GA — https://visualstudiomagazine.com/articles/2026/01/22/copilot-studio-extension-for-vs-code-goes-ga.aspx
- Praneet S — Copilot Studio Deep Dive — https://medium.com/@praneetsy/copilot-studio-deep-dive-a2a086e22f73
- Khushboo Nijhawan — Building Agents in Copilot Studio (the guide I wish existed) — https://medium.com/@khushboo.nijhawan/building-agents-in-copilot-studio-heres-the-guide-i-wish-existed-when-i-started-f31c33bc640a
- Mohit Aggarwal — What I learned about Topics — https://medium.com/product-powerhouse/i-built-a-copilot-studio-agent-that-actually-worked-heres-what-i-learned-about-topics-1b521d310fee
- peafowlit — Multi-Agent Orchestration: a practical architecture guide — https://medium.com/@peafowlit/multi-agent-orchestration-with-copilot-studio-a-practical-architecture-guide-f655597371b1
- George Karapetyan — 6 practical ways to handle user-uploaded files — https://medium.com/@georgekar91/6-practical-ways-to-handle-user-uploaded-files-in-copilot-studio-a729f06642be
- Building Autonomous AI Agents: a deep dive into Copilot Studio's full experience — https://medium.com/codex/building-autonomous-ai-agents-a-deep-dive-into-copilot-studios-full-experience-687b553ea7a8
- Matthew Devaney — Build a Copilot Studio agent with generative orchestration — https://www.matthewdevaney.com/how-to-build-a-copilot-studio-agent-with-generative-orchestration/

> These practitioner posts are captured in [guidance/field-notes-practitioner-tips.md](guidance/field-notes-practitioner-tips.md) and [reference/file-and-attachment-handling.md](reference/file-and-attachment-handling.md). Treat as secondary — verify specifics against current product behavior.

> Note on freshness: Copilot Studio ships monthly; the new experience and multi-agent connections are in preview. Always re-verify `kind`s and commands against the installed extension version and the shipped `bot.schema.yaml-authoring.json` when building.
