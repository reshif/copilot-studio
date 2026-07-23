# 02 — New vs Classic Agent Experience

Copilot Studio currently offers **two authoring/runtime experiences**. You choose which one when you **create** an agent, and **you cannot convert between them** in either direction.

## Classic experience

- **Topic/flow-first.** You author explicit conversation **topics**, flows, and branching logic.
- Orchestration behavior is **configurable** (you can turn generative orchestration on/off, tune settings).
- Fully supported and still the basis of most existing production agents.
- Docs entry points: "Create and delete agents", "AI-based agent authoring overview" (`nlu-gpt-overview`).
- This is the experience whose YAML the VS Code extension currently clones/edits (topics as `AdaptiveDialog`, etc.).

## New experience (production-ready preview)

A **redesigned authoring and runtime environment** built on an **enhanced orchestration runtime** that replaces the classic orchestration model.

Key differences:

- **Natural-language-first.** Instead of authoring explicit topics and branching, you **describe the agent in natural language** and the system generates the underlying configuration.
- **Unified single-surface design** (one place to build).
- **Enhanced orchestration for *all* agents** — not configurable/toggleable like classic. Supports **deep reasoning** and higher-quality responses, particularly over **Microsoft 365 data**.

### The new-experience Build tab organizes an agent around:

- **Instructions** — identity, tone, scope, behavior.
- **Knowledge** — connected data sources, org data via **Microsoft IQ**, and **memory**.
- **Tools and skills** — tools = actions (call APIs, run flows); skills = reusable structured behaviors.
- **Model** — pick the LLM powering reasoning.
- **Connected agents** — delegate specialized tasks to other agents.

### New-experience authoring surface (tabs)

| Tab | Purpose |
|-----|---------|
| **Build** | Configure identity, knowledge, tools, skills, model. |
| **Preview** | Test the agent interactively in a preview chat. |
| **Evaluate** | Create and run **test sets** to measure agent quality. |
| **Monitor** | Review recent tasks, files accessed, and activity. |

### New-experience agent lifecycle

1. **Create** — from the home page, choose the new experience.
2. **Build** — configure who the agent is, what it knows, what it can do, and its limits.
3. **Test** — use **Preview** and **Evaluate** tabs.
4. **Publish** — deploy to channels.
5. **Monitor** — track tasks, activity, performance post-deployment.

## Important caveats

- The new experience is a **production-ready preview** (new **workflows** experience is public preview). Some capabilities available in classic are **not yet** in the new experience.
- **No conversion** between experiences in either direction — pick deliberately.
- Public-preview features aren't for production and may have restricted functionality; subject to supplemental terms of use.

## How to try it

On the classic home page banner, select **Try it now**. Toggle back anytime with the **New experience** toggle.

## Practical guidance for our framework

- For **code-first / VS Code work today**, we're primarily operating against the **classic** YAML model (topics as `AdaptiveDialog`, `actions/`, `knowledge/`, `trigger/`), which is what the extension clones. Generative orchestration can still be enabled on a classic agent.
- Design agents so the **descriptions** (topic `modelDescription`, tool names/descriptions, knowledge source descriptions) are high quality — the orchestrator routes on these regardless of experience.
- Track the new experience for eventual migration, but don't assume feature parity yet.

Sources: see [12-glossary-and-sources.md](12-glossary-and-sources.md).
