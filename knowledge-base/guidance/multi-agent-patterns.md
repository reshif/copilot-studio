# Guidance — Multi-Agent Patterns & Best Practices

Generative orchestration supports multi-agent systems (one agent calling others). Splitting a problem into specialized agents makes it modular, scalable, and manageable — but adds latency, governance, and testing overhead. See also [../09-multi-agent-orchestration.md](../09-multi-agent-orchestration.md) for mechanics.

## Two composition styles

**Inline agents (child agents)** — small reusable workflows *within* the same agent; often just topics used as subroutines (e.g. a "Translate Text" step). They **share context** with the main agent, so passing data is simple. *Best practice: single responsibility, well tested.*

**Connected agents** — separate agents with their **own orchestration, tools, knowledge** (and possibly different privileges). The main agent delegates part of a request (e.g. IT agent calls a Sales agent for pricing). They enable modularity/domain separation and can **bypass plan limits**, but need governance.

## When to make a SEPARATE (connected) agent
Only if the subtask:
- Is complex enough to need **its own tools/knowledge** (different domain).
- Needs **different governance/access controls** than the main agent.
- Is **reusable** across many main agents (a "service agent").

Otherwise use an inline agent — simpler, less overhead. **Start with one agent; split only when you see a real need or a boundary a single agent shouldn't cross.**

## Governance for connected agents
- **Orchestration:** give the parent clear hand-off criteria; describe the connected agent's purpose well (treat it as an agentic "tool" with a description).
- **Data handoff:** Copilot Studio passes **conversation history by default** to the connected agent; pass specific parameters too (e.g. the already-known user name) to avoid re-asking.
- **Security:** a connected agent may have access the parent lacks — don't let a parent call it in ways that bypass restrictions (e.g. parent can't delete records but connected agent can). Treat a connected-agent call like any powerful action; gate sensitive ones with checks/consent.
- **Audit/monitoring:** log when a connected agent is invoked and what it did; correlate parent + connected transcripts via telemetry identifiers.

## The 9 rules for multi-agent instructions (Microsoft)

1. **Single response principle** — only ONE agent talks to the user per turn (the parent). Parent instruction: *"You're the only agent that communicates with the user. Combine findings from all child agents into a single response."* Subagents are researchers, not responders.
2. **Subagents must declare their role** — they don't know they're subagents. Every subagent: *"You're a subagent. Do NOT reply to the user directly. Search for information and return findings to the parent agent, which handles all user communication."*
3. **Clear, directive language** — MUST/DO NOT/NEVER/ONLY. Soft phrasing loses priority. *"NEVER reply to the user directly. ONLY return your findings."*
4. **One knowledge source per subagent (no overlap)** — distinct, nonoverlapping sources. If you only have one source, use a single agent — multi-agent adds value only when sources genuinely differ.
5. **Accurate, distinct subagent descriptions** — the parent routes on them. *CA-1: "Searches HR policy documents..."; CA-2: "Searches IT knowledge base..."* Never identical/generic.
6. **Parent defines the orchestration pattern** — explicit: *"1. Invoke both child agents. 2. Wait for both to return findings. 3. Combine into a single unified response. 4. Deliver exactly one response. Child agents must not reply directly."*
7. **Reinforce "no direct reply" in the delegated task** — parent adds to each delegation: *"Return your findings only. Don't reply to the user."*
8. **Test with domain-mismatch queries** — ask things outside all subagent domains (weather when agents do HR/IT); verify graceful "no info found" and that the parent handles "both found nothing."
9. **Prefer ASK over INFORM for follow-ups** — "ask" = two-way (stay with the subagent); "inform" = one-way (user reply goes back to the parent planner as a brand-new query). Use ask when you expect a reply.

### Quick checklist
- [ ] Parent: "only I respond to the user."
- [ ] Every subagent: "don't reply to the user directly."
- [ ] Strong directive language (MUST/NEVER/ONLY).
- [ ] Each subagent has a unique, nonoverlapping knowledge source.
- [ ] Subagent descriptions accurate, distinct, specific.
- [ ] Parent defines full pattern (invoke → wait → combine → respond).
- [ ] Parent passes "no direct reply" in delegated tasks.
- [ ] Tested with domain-mismatch queries.
- [ ] Ask vs inform used correctly.

## Cost & limits reminders
- Each orchestration hop adds latency and its own credit consumption (see [../reference/licensing-and-credits.md](../reference/licensing-and-credits.md)).
- Single-agent routing degrades past ~30–40 combined choices; each agent's orchestrator maxes at 128 tools (recommend ≤25–30).
- Connectable external agents: same-env Copilot Studio, A2A, Microsoft Foundry, Fabric Data, Microsoft 365 Agents SDK (see [../09-multi-agent-orchestration.md](../09-multi-agent-orchestration.md)).
