# Guidance — Instructions, Prompting & Descriptions (the craft)

This is the single highest-leverage skill for agent quality. Under generative orchestration, **instructions + descriptions ARE the program.** Microsoft explicitly says: *the system treats agent instructions like code — wrong "code" breaks the system.*

## What instructions do
Instructions (on the agent **Overview** page) drive three things:
1. **Selection** — which tool/knowledge/topic/agent to call for a query or trigger.
2. **Input filling** — values for tool inputs from context.
3. **Response generation** — the final summarized answer.

**Grounding rule:** an agent can only act on instructions for capabilities it actually has. Add the tool/knowledge/agent first, then reference it. "Search the FAQ" does nothing unless that FAQ is a configured knowledge source.

## Referencing objects with `/`
While typing instructions, type `/` to insert a reference to a **knowledge source, tool, topic, agent, variable, or Power Fx expression**. Use the **exact** object name (slight naming differences hurt results). You generally *don't* need to list available tools — the agent already knows them from names/descriptions. Use `/` only to disambiguate or force a specific sequence.

## Vocabulary that works (from Microsoft)
Use verbs like **Get/Use** for retrieving/parsing; **From/With** for acting on results.

| Goal | Words |
|------|-------|
| Conditions | when, if, ensure, compare |
| Filter | from, include, exclude, compare, identify |
| Data | provide, retrieve, get, use, analyze, extract |
| Tools | notify, direct, ask, assign |

Use **strong directive language** — **MUST, DO NOT, NEVER, ONLY**. The platform injects system instructions in strong language; soft phrasing ("please try to", "ideally", "you should") loses priority in conflicts.

## Structure: Constraints → Response format → Guidance
Microsoft's recommended conversational-instruction shape (combine all three):
```text
# Constraints
Only respond to requests about educational, legal, wellness, health, dental, and newborn
benefits for employees and dependents.

# Response format
Provide benefit types with details and health-plan comparisons in a table. Add a column for
available options. Include provider details and an enrollment link. Use bold/underline as needed.

# Guidance
Search only within the country folder relevant to the employee's country.
```
Number/bullet steps and say the agent must follow them **in order**. Markdown improves readability and model parsing.

## Specific techniques
- **Route hints** (only when ambiguous):
  ```text
  Use the FAQ documents only if the question is not about Hours, Appointments, or Billing.
  Only use the ticket-creation topic for creating tickets.
  For other fix-it requests, use the troubleshooting topic.
  ```
- **Force a sequence/tool** (helpful when >5 tools): "When the user provides their preferred laptop, create a purchase order using /**Purchase Order**."
- **Don't reach out**: "Don't ask the user for any details."
- **Input hints**: "Use the email address from the contact field of the lead when drafting the follow-up email."
- **Guardrails / scope**: "Only respond to messages relevant to Contoso and ordering coffee. Otherwise, tell the user you can't help."
- **Format**: "Always give order-status responses in a table." / "Send emails using rich text formatting." (also add to the tool description.)
- **Tone**: default is professional/polite — only add tone instructions for special cases.

## Follow-up questions (powerful pattern)
Instead of authoring every branch, tell the agent to ask the right next question from what it can do/know:
- Reference tools/knowledge/variables in instructions so the agent forms context-aware follow-ups.
- Ask it to conclude responses with relevant follow-ups.
- Keep examples visible so it learns the pattern.
- **Caveat:** follow-up questions **require Allow ungrounded responses = ON** — otherwise the orchestrator treats an uncited clarifying question as ungrounded and suppresses it (defaults to the fallback message).

## CITATIONS — do not touch (critical)
**Never** write instructions that modify/override/suppress citations, or use the words "citation"/"reference" to reshape them. If you break citation markers, the orchestrator may treat a grounded answer as model knowledge and **discard it** when *Allow ungrounded responses* is off — looks like the agent is broken. To make grounded answers reliable: instruct "always include an in-text citation for every statement," and avoid rigid formats ("JSON only") that strip citation markers.

## Security — prompt-injection / jailbreak defense
Triggers and untrusted knowledge (emails, tickets) can carry attacker instructions. Defend in the agent instructions:
- Limit what the agent may do after reading knowledge ("only email after checking a knowledge source for context").
- Limit tool parameters ("only email this specified list of individuals").
- If content filtering blocks legitimate behavior, state that the behavior is expected.

## Autonomous / trigger instructions
- Triggers are edited in **Power Automate**, not Copilot Studio. Trim the payload (send only needed fields, e.g. the email subject) to reduce size.
- Tell the agent what to do with the payload ("Onboard the following employee:") followed by the trigger body; it then follows the main onboarding instructions.
- You can add **multiple triggers**, each with its own instructions, for different use cases in one agent.

## What instructions CANNOT do (use topics/config instead)
- Can't change how **Adaptive Cards** trigger → edit the card / its trigger phrases.
- Can't override the **default fallback message** → edit the **Fallback** system topic.
- Can't modify **search retrieval** logic or how retrieved docs are **shared** → remove such instructions.
- **Multilingual** via instructions is not officially supported/tested — validate before promising.
- Avoid **ambiguous terms** ("typing box") — unpredictable.

## Debugging instructions
If complex instructions yield no/blocked responses: **remove all instructions, then add back one at a time, testing between each.** Keep instructions **short and simple** — for summarization/conversational flow, not system-level behavior.

## Prompt-tool best practices (the `Prompt` tool / AI Builder prompts)
Prompts are single-turn model calls (see [../reference/prompts-and-agent-flows.md](../reference/prompts-and-agent-flows.md)). Best practices:
- **Be specific**; avoid vague/ambiguous wording.
- **Use examples** to shape output.
- **Keep it simple** and **brief** — long prompts cause latency/timeouts.
- **Give a way out**: "respond with 'not found' if the answer isn't present" (reduces hallucination).
- **Test and refine** with sample data.

## Description authoring (recap — drives routing)
Every tool/topic/knowledge/agent needs a **distinct, specific** description; the orchestrator routes on name+description (+ input/output names). Say what it does, when to use it, and **what it can't do**. Avoid vague ("can answer questions") and jargon (spell out acronyms). See the worked good/bad examples in [../reference/generative-orchestration.md](../reference/generative-orchestration.md#authoring-descriptions-the-single-most-important-skill).

## Key takeaways (Microsoft's own)
- Keep instructions **as simple and short as possible**.
- Use instructions for **summarization and conversational flow**, not system behaviors.
- Use **topic configuration** for fallback and Adaptive Card customization.
- **Validate** features (e.g. multilingual) before promising them.
