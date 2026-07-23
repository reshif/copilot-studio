# Reference — Agent Memory (preview)

**Memory** (new agent experience, preview) lets an agent remember context and patterns across a user's interactions to personalize responses and work more efficiently. Memories are **per-user and never shared** between users.

## How it works (3 stages)
1. **Capture** — the agent records signals: user preferences, observed patterns, notes about its own work.
2. **Store** — signals are saved as **files in the user's per-agent memory folder**, in a **tenant-scoped** store.
3. **Apply** — on later interactions the agent reads that memory to inform responses/decisions.

## Key behaviors
- **Per-user store** per agent (isolation between users).
- **28-day expiry**: if a user has no activity with the agent for 28 days, their memories for that agent are deleted.
- **Toggle**: control via the **Memory** toggle in the components panel. Turning it **off doesn't delete** stored memories — it just stops the agent using them.

## When to use
- Assistants that benefit from remembering preferences/working style across sessions (e.g. "always summarize in bullet points", preferred region/product).
- Reduces re-asking and makes multi-session workflows smoother.

## Design & governance considerations
- It's **preview** — validate before production; treat as subject to change.
- Memory is another place user data lives → factor into **privacy/compliance** review (see [governance-security-dlp.md](governance-security-dlp.md)).
- Don't rely on memory for correctness-critical facts — it's personalization, not a system of record.
- Distinct from **conversation history** (short-term planner context; see [generative-orchestration.md](generative-orchestration.md)) and from **knowledge** (grounding sources).

> This is part of the **new agent experience**; the VS Code extension currently centers on classic-style definitions, so verify memory support/representation against the shipped schema when building code-first.
