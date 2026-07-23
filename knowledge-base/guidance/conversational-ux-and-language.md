# Guidance — Conversational UX & Language Understanding

How the agent *feels* to talk to, and how it actually understands people.

---

## Part 1 — Conversational UX (CUX) principles

CUX flips the usual relationship: instead of users learning the system's language (syntax, menus, information architecture), **the system understands the user's natural language** — including speech patterns, colloquialisms, chit-chat, and misspellings.

> **Conversation is personal.** Users assign a personality to anything that responds to them, and that effect is *stronger* in natural language. Research shows users become more attached to devices they interact with conversationally. CUX designers therefore have a responsibility to **honor the emotional state** of the person on the other end.

### The five principles

| Principle | What it means | Design implication |
|-----------|---------------|--------------------|
| **Efficiency** | Natural language is often the fastest path ("Set an alarm for 8 AM" vs navigating an app) | Identify your users' **most common questions** and answer them in the fewest turns |
| **Accessibility** | Disability is a mismatch between person and environment — situational, temporary, or permanent | Voice for hands-busy/eyes-busy or limited mobility; images/video as alternative presentation |
| **Intuitiveness** | Users discover *how* without knowing the steps ("Connect my headset", "I can't print") | Let intent + dialog design guide them; don't require technical vocabulary |
| **Empathy** | Conversations are emotional even when users know it's an agent | Acknowledge frustration; use careful language and positive reinforcement; respect linguistic/cultural context |
| **Trust** | A consistent, appropriate **persona** signals reliability and care | Even with no name or avatar, language conveys a persona — craft it deliberately and keep it **consistent** |

**Chit-chat** is a legitimate tool: a few well-chosen casual exchanges build rapport, teach users how to talk to the agent, and **smooth over the inevitable errors** elsewhere.

> When a CUX responds intelligently, the user feels **heard**; when the response carries relevant emotional nuance, they feel **understood**. That's the foundation of trust.

### Practical CUX checklist
- [ ] Persona defined (tone, formality, humor level) and **consistent** across topics.
- [ ] Greeting sets expectations, including **AI disclosure**.
- [ ] Most common intents answered in minimal turns.
- [ ] Fallback is helpful, not a dead end.
- [ ] Escalation path is obvious and dignified.
- [ ] Error/reprompt messages explain what to do next.
- [ ] Accessibility validated (voice, screen readers, plain language).
- [ ] Emotional edge cases (frustration, complaints) handled with care.

---

## Part 2 — Language understanding (LU)

An **utterance** is what the user types/says. The agent decomposes it into **intent** + **entities**, using context.

LU covers: **intent recognition** ("book a flight"), **entity extraction** ("Paris", "next week"), **context awareness** (pronouns/references across turns), and **handling ambiguity** ("bank" = financial vs river).

### The three LU approaches

| | **Generative orchestration** (default) | **Built-in NLU** (classic) | **Azure CLU** (custom) |
|---|---|---|---|
| **Key features** | LLM-based; handles **multi-intent** utterances; chains topics/actions/knowledge; **auto-generates questions** for missing inputs; allows corrections mid-run; produces a unified answer | Pretrained model with predefined entity types; configured via **trigger phrases** + custom entities (closed list/regex) | More languages with native models; customizable intent model for accuracy/industry vocabulary; advanced entity extraction (same-type, silent extraction) |
| **Limits** | **5 messages per topic/action chain**; **128 topics/actions** for triggering; English-only | **Single intent** per query; can't be extended; same-type multi-entity slot filling needs disambiguation | **Single intent** per query; Azure config + extra cost; Azure service limits; **CLU intents and CS topics must be kept in sync** |

**NLU+** is the high-accuracy option for large enterprise agents (many topics/entities, lots of training samples). For **voice-enabled agents, NLU+ training data also optimizes speech recognition.**

### Topic structure has shifted
Topics moved from rigid intent-based paths to **modular instructions the orchestrator calls**. Generative orchestration handles most routing dynamically; **topics provide structured fallback when precision is needed**. Keep: entry-point topics, reusable subtopics, **disambiguation topics**, and fallback/conversational-boosting as safety nets.

See [../reference/generative-orchestration.md](../reference/generative-orchestration.md), [agent-design-playbook.md](agent-design-playbook.md) (trigger phrases + slot filling).

---

## Part 3 — Multilingual & localization

Everything keys off the system variable **`System.User.Language`** — the central control point for language behavior.

**What it drives:**
- **Knowledge search in the user's language** — the query is auto-translated to the language in `System.User.Language`.
- **Response generation in the user's language** — regardless of the question's language or the source documents' language.
- **Manual override** — set it explicitly to force a language (useful for testing or controlled scenarios).

**Auto-detection**: a trigger starts a language-detection flow on message receipt → sets `System.User.Language` → the conversation continues in that language for both retrieval and generation. (Microsoft publishes an *Auto Detect Language for Generative Responses* sample in `CopilotStudioSamples`.)

### Three multilingual architectures
1. **Separate agents per language** — max control, max maintenance.
2. **Single multilingual agent with pre-authored translations** — balanced; use localization files (JSON/ResX).
3. **Real-time multilingual** — translation services between user and agent; least authoring, most runtime dependency.

Choose based on: usage, separation concerns, scale, update cadence, available resources.

### Localization best practices
- Define **primary and secondary languages**; use localization files for prompts/messages/topics.
- **Test multilingual scenarios** explicitly — simulate interactions per language.
- **Use auto-translation** for breadth, but hand-author translations for **critical or nuanced content**.
- **Monitor and refine** with analytics on language usage.

> ⚠️ Two cautions: generative **orchestration** is English-only, and *instructions asking for multilingual behavior aren't officially tested or supported* — validate before promising it. Copilot Studio publishes in 20+ languages, but verify per feature.

---

## Common technical challenges
Keeping **Azure CLU intents in sync** with Copilot Studio topics · handling **ambiguous utterances** · scaling **multilingual deployments**. Mitigate with fallback configuration, **bulk trigger-phrase testing** (Copilot Agent Kit), and relay-based translation services.
