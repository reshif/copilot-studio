# Microsoft Copilot Studio — Local Knowledge Base

A working reference for building **Copilot Studio agents** using the **Copilot Studio extension for Visual Studio Code** (code-first, YAML-based), plus everything needed to understand the platform end-to-end.

> Purpose: be proficient enough with the framework that when we start building agents in VS Code with the Copilot Studio extension, we already understand the full picture — the concepts, the file/YAML schema, the authoring building blocks, the dev workflow, and ALM/deployment.

Last researched: **2026-07-23**. Sources are Microsoft Learn (official docs), the Microsoft 365 Developer Blog, and the `microsoft/skills-for-copilot-studio` code-first framework. See [12-glossary-and-sources.md](12-glossary-and-sources.md) for the full source list.

---

## How to read this KB

Read roughly in order. Files 01–02 give you the mental model; 03 is the VS Code tooling; 04–09 are the actual building blocks you'll write in YAML; 10 is shipping; 11 is our reusable framework strategy.

| # | File | What it covers |
|---|------|----------------|
| 01 | [01-copilot-studio-fundamentals.md](01-copilot-studio-fundamentals.md) | What Copilot Studio is, agent types, where it sits in the Microsoft stack |
| 02 | [02-agent-experience-new-vs-classic.md](02-agent-experience-new-vs-classic.md) | New (generative, natural-language-first) vs classic (topic/flow) experience |
| 03 | [03-vscode-extension.md](03-vscode-extension.md) | The VS Code extension: install, auth, clone, edit, sync, commands |
| 04 | [04-agent-definition-yaml-schema.md](04-agent-definition-yaml-schema.md) | Cloned folder layout, `.mcs.yml` files, `kind` values, schema |
| 05 | [05-topics-and-dialogs.md](05-topics-and-dialogs.md) | `AdaptiveDialog`, triggers, node/action kinds, YAML examples |
| 06 | [06-tools-and-actions.md](06-tools-and-actions.md) | Tools: connectors, flows, prompts, REST, MCP; the `actions/` folder |
| 07 | [07-knowledge.md](07-knowledge.md) | Knowledge sources, RAG, generative answers, files |
| 08 | [08-power-fx-and-variables.md](08-power-fx-and-variables.md) | Variable scopes, Power Fx syntax, common formulas |
| 09 | [09-multi-agent-orchestration.md](09-multi-agent-orchestration.md) | Child agents, connected agents, A2A, orchestration patterns |
| 10 | [10-alm-publishing-testing.md](10-alm-publishing-testing.md) | Solutions, environments, publishing/channels, testing/evaluation |
| 11 | [11-code-first-framework-and-workflow.md](11-code-first-framework-and-workflow.md) | `skills-for-copilot-studio`, our reusable build framework & conventions |
| 12 | [12-glossary-and-sources.md](12-glossary-and-sources.md) | Glossary + all source URLs |

### In-depth reference library (`reference/`)

The `reference/` folder is the deep, build-ready detail — full property tables, every node/entity/variable/limit. Use the 01–12 files to learn concepts; use these when authoring.

| File | What it covers |
|------|----------------|
| [reference/node-types.md](reference/node-types.md) | Every topic node (Message, Question, Condition, SetVariable, Parse, Loop, Redirect, End, HTTP, Authenticate, generative answers) + YAML kinds + properties + system topics |
| [reference/variables-and-entities.md](reference/variables-and-entities.md) | Variable types/scopes, **full system-variable table**, env vars, auth vars; prebuilt entity→type table, custom (closed/regex/open), slot filling |
| [reference/generative-orchestration.md](reference/generative-orchestration.md) | Generative vs classic, how selection works, **authoring descriptions**, control nodes, limitations, tool-count guidance |
| [reference/tools.md](reference/tools.md) | Tool types (connector/flow/prompt/REST/MCP/CUA), inputs/outputs, completion, end-user vs maker auth, limits |
| [reference/knowledge.md](reference/knowledge.md) | Source table, classic vs generative search, ungrounded/web/moderation/semantic-search settings, citations, limits |
| [reference/triggers.md](reference/triggers.md) | All trigger types, priority order, conditions, autonomous/workflow triggers |
| [reference/publishing-security-limits.md](reference/publishing-security-limits.md) | Publish flow, channel list & experience, auth options, quotas & all numeric limits |
| [reference/prompts-and-agent-flows.md](reference/prompts-and-agent-flows.md) | Prompt tool (AI Builder prompts, tiers/billing) + agent flows (vs cloud flows, triggers, enforcement, convert) |
| [reference/adaptive-cards.md](reference/adaptive-cards.md) | Adaptive Cards: schema versions per channel, JSON/Power Fx, inputs→variables, reprompt, consecutive-card pitfalls |
| [reference/computer-use.md](reference/computer-use.md) | Computer Use (CUA): models (incl. Claude), credentials, access control, per-step billing, safety |
| [reference/analytics-evaluation.md](reference/analytics-evaluation.md) | Analytics (sessions, CSAT, outcomes, themes, transcripts, App Insights) + evaluation test-set methods |
| [reference/licensing-and-credits.md](reference/licensing-and-credits.md) | Copilot Credits economics: billing rates, reasoning-model meters, examples, 125% overage & flow enforcement |
| [reference/governance-security-dlp.md](reference/governance-security-dlp.md) | Security controls, DLP data policies (connectors/endpoint filtering), environment strategy |
| [reference/memory.md](reference/memory.md) | Agent Memory (preview): capture/store/apply, per-user, 28-day expiry, toggle |
| [reference/agent-types-and-m365-extensibility.md](reference/agent-types-and-m365-extensibility.md) | Custom vs declarative vs custom-engine vs SDK agents; M365 Copilot extensibility; templates & Agent Library |
| [reference/voice-and-telephony.md](reference/voice-and-telephony.md) | Voice/IVR: basic vs real-time, enable voice, timeouts/barge-in/silence, SSML, DTMF, transfer, voice variables |
| [reference/extensibility-mcp-skills-a2a.md](reference/extensibility-mcp-skills-a2a.md) | MCP servers, Bot Framework/Agents SDK skills (manifest/limits), Agent2Agent (A2A) protocol |
| [reference/dataverse-knowledge.md](reference/dataverse-knowledge.md) | Dataverse-as-knowledge: prerequisites, synonyms/glossary tuning, multiline/file columns, limits |
| [reference/tooling-coe-and-agent-kit.md](reference/tooling-coe-and-agent-kit.md) | Power CAT Copilot Agent Kit (batch testing, rubrics, governance hubs) + CoE Starter Kit |
| [reference/file-and-attachment-handling.md](reference/file-and-attachment-handling.md) | The native "agents can't process uploads" gotcha + a no-code→full-custom solution ladder |
| [reference/custom-knowledge-sources.md](reference/custom-knowledge-sources.md) | The **YAML-only `OnKnowledgeRequested` trigger**: bring your own search endpoint, system variables, full worked example, 15-snippet cap |
| [reference/metrics-and-roi.md](reference/metrics-and-roi.md) | The full **metrics library** (7 categories), four value drivers, the **Agent Assisted Hours/Value formulas** with a worked example, leading vs lagging indicators, six adoption levers |
| [reference/first-agent-build-guide.md](reference/first-agent-build-guide.md) | **Capstone**: end-to-end worked build (full YAML) + pre-flight checklist + pitfalls |

### Guidance (best-practice / decision-making) (`guidance/`)

Craft and judgment — how to make good choices, not just what exists.

| File | What it covers |
|------|----------------|
| [guidance/agent-design-playbook.md](guidance/agent-design-playbook.md) | ⭐ **START HERE to build**: the what/when/where/how decision playbook — design gate, platform choice, orchestration mode, **the 3 control layers**, component decision matrix, autonomy, agent flows, testing loop, throughput planning, golden rules & anti-patterns |
| [guidance/instructions-prompting-and-descriptions.md](guidance/instructions-prompting-and-descriptions.md) | **The craft**: writing agent instructions (vocabulary, structure, guardrails, follow-ups, jailbreak defense, citation rules, debugging), prompt-tool best practices, description authoring |
| [guidance/multi-agent-patterns.md](guidance/multi-agent-patterns.md) | Inline vs connected agents, when to split, the **9 multi-agent instruction rules** + checklist |
| [guidance/plan-scope-and-value.md](guidance/plan-scope-and-value.md) | Before the build: objectives & KPIs, **ROI/business-value framework**, solution architecture, Well-Architected pillars, responsible AI |
| [guidance/conversational-ux-and-language.md](guidance/conversational-ux-and-language.md) | The 5 **CUX principles**, persona & chit-chat, the 3 language-understanding approaches (generative / NLU / CLU), multilingual & localization |
| [guidance/govern-operate-and-improve.md](guidance/govern-operate-and-improve.md) | **Zoned governance** (3 zones), security roles & sharing rules, control levels, **ALM golden rules + non-solution-aware items**, the 8 test types, engagement/outcome/deflection metrics, evaluation triage |
| [guidance/integration-and-performance-testing.md](guidance/integration-and-performance-testing.md) | Choosing connectors vs HTTP vs agent flows vs skills, integration planning table, **load-test plan with concrete targets**, custom analytics/Synapse export |
| [guidance/adoption-maturity-model.md](guidance/adoption-maturity-model.md) | The **5 maturity levels × 5 pillars** matrix, how to find your binding constraint, signals you're stuck at pilot |
| [guidance/field-notes-practitioner-tips.md](guidance/field-notes-practitioner-tips.md) | **Community field notes** (secondary): descriptions-as-code, topic over-segmentation, cost-aware orchestration, the compounding test loop, "start with one agent" |

---

## The one-paragraph mental model

Copilot Studio is Microsoft's platform for building **AI agents** (conversational + autonomous) on top of Power Platform / Dataverse. An agent is defined by **instructions** (identity/behavior), **knowledge** (grounding data), **tools** (actions it can take), a **model** (the LLM), **topics** (scripted dialog flows), **triggers** (when things activate), and optionally **connected/child agents**. Historically you built this in a web canvas; now the **VS Code extension** lets you **clone the full agent definition to disk as YAML** (`.mcs.yml`), edit it with IntelliSense + AI assistants (GitHub Copilot / Claude Code), version it in Git, and **sync (apply) changes back** to a cloud environment for testing. That code-first path is what makes a **reusable, repeatable build framework** possible.

## Fast path to "building"

1. Install extension → sign in → pick environment (see [03](03-vscode-extension.md)).
2. Create a minimal agent in the web portal, then **Clone** it to a local folder.
3. Learn the folder layout & `agent.mcs.yaml` (see [04](04-agent-definition-yaml-schema.md)).
4. Add/modify **topics, tools, knowledge** in YAML (see [05](05-topics-and-dialogs.md)–[07](07-knowledge.md)).
5. **Apply** changes → **test** in the Copilot Studio test pane → iterate.
6. Commit to Git, use PR review, promote via solutions/pipelines (see [10](10-alm-publishing-testing.md)).
