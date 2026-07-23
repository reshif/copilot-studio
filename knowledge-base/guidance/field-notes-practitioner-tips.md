# Guidance — Field Notes (practitioner tips from the community)

> ⚠️ **Secondary sources.** These are hard-won lessons from Medium deep-dives and practitioner blogs (not official Microsoft docs). They align with and sharpen what's in the official reference, but **verify against current product behavior** before relying on specifics. Sources listed at the bottom and in [../12-glossary-and-sources.md](../12-glossary-and-sources.md).

Consistent themes across every good practitioner write-up:

## 1. Descriptions are the real source code
- The orchestrator **doesn't read your branch structure — it reads your descriptions.** Write tool/knowledge/topic/agent descriptions with the care of function documentation. **Vague description = tool that never fires.** (Matches official [generative-orchestration.md](../reference/generative-orchestration.md), but practitioners rank this as *the* #1 determinant of quality.)
- Put **tool-usage rules in the agent Instructions, not the Tools tab.** Without explicit instruction-level rules, the model calls tools **excessively, never, or with invalid data.**

## 2. Instructions: shorter is better
- Practitioner target: **~1,500–3,000 characters**; quality **degrades well before** the hard cap. "Shorter instructions = better copilots."
- Must-have sections: **Identity, Scope, Knowledge-usage rules, Tool-usage rules, Trigger behavior, Tone, Fallback behavior.** Cut verbose storytelling, repeated tone declarations, and restated rules.
- Debugging: if complex instructions break responses, **strip to nothing and add back one rule at a time** (also the official method).

## 3. Tight knowledge scoping beats broad coverage
- **3 relevant sources > 30 vague ones.** Precision of grounding matters more than volume (consistent with the 25-source weighting and description-based filtering in [knowledge.md](../reference/knowledge.md)).
- **Conflicting/old files cause inconsistent answers** — the agent grounds on what you gave it, so **delete stale versions** and **re-upload after edits** to force re-indexing.
- **RAG quality is downstream of document hygiene**: clean text (not image-only PDFs), clear headings, descriptive filenames, **split big PDFs into smaller files.** Rough indexing times cited: 1–5 pp ≈ 1–3 min, 5–25 pp ≈ 4–30 min, 25+ pp ≈ 30 min+.
- Test grounding with **direct, indirect, ambiguous, and multi-step** questions — not just the happy path.

## 4. Topics: model INTENT, not phrasings (the biggest beginner mistake)
- Real failure story: an HR agent with **34 topics** each capturing a slight wording variation of "leave policy" → **contradictory responses, missed triggers, confused flows.** Fix took two weeks of consolidation.
- **Don't** create a topic per phrasing variation; **don't** assume tutorial phrases mirror real users; **don't** skip testing with unscripted input.
- **Do** consolidate by **semantic intent**, **audit topics specifically when you see generic fallback replies**, and **test with realistic, unprompted language.**
- Keep individual topics **small**; lean on **conditions** for branching rather than heavy per-topic logic; reserve topics for **repetitive tasks, structured data collection, triage.**

## 5. Choose orchestration deliberately (cost/reliability)
- Generative orchestration **reasons every turn → costs more tokens/credits.** For **high-volume, predictable intents**, deliberately use **classic topic-trigger** matching to preserve cost and add determinism. (Cross-ref [licensing-and-credits.md](../reference/licensing-and-credits.md): classic answer = 1 credit vs agent action = 5.)
- Blend: generative for open-ended routing, authored topics for the deterministic/compliance paths.

## 6. Start with one agent — earn complexity
- "**Start with one agent, one job. Every orchestration layer you add is a layer you have to debug.**" Only split into child/connected agents when routing genuinely degrades or a governance/ownership boundary demands it (matches [multi-agent-patterns.md](multi-agent-patterns.md)).
- Multi-agent architecture vocabulary practitioners use: **Embedded** (child) agents for rapid single-team iteration; **Connected** agents when action count exceeds ~30–40 or business units need shared capability; **MCP-connected tools** for cross-ecosystem actions.
- "**A well-scoped agent with 10–15 focused actions outperforms a bloated agent with 60+.**" Keep the **orchestrator lean = a router, not an executor** — don't duplicate child-agent capability in the parent.
- Example service-desk pattern: one orchestrator → IT agent (Dataverse+ServiceNow), HR agent (SharePoint), Finance agent (Fabric Data) — each **independently owned/tested**, orchestrator handles routing + human escalation on low confidence. Composable: add domains without rebuilding the orchestrator.

## 7. The testing loop that actually compounds quality
- **Ship → watch analytics → turn weird real conversations into test cases → fix → re-run the suite → repeat.** Agent quality compounds like code quality with a good test suite.
- **Convert real analytics transcripts into regression test sets early** — teams whose agents improve over time simply **automated regression testing on real edge cases** before scaling. (Pair with [analytics-evaluation.md](../reference/analytics-evaluation.md) test methods and the [Copilot Agent Kit](../reference/tooling-coe-and-agent-kit.md) batch testing.)
- The **activity map** in the test pane is the primary debugger — it shows exactly which knowledge/tool/topic the orchestrator picked per turn; use it to catch routing failures **before** production.
- Evaluation tab AI scores are useful but **keep a human in the loop.**

## 8. Governance is the production gate, not an afterthought
- Auth gaps, DLP impacts, and publishing restrictions **surface in the authoring experience** and block risky deploys — treat them as a gate, not a nuisance (see [governance-security-dlp.md](../reference/governance-security-dlp.md)). "Your agents are only as intelligent as the data they can reach" — security trimming/context access must align with agent governance. Build **observability/tracing from day one** for multi-system flows.

## 9. Copilot Studio does NOT natively process user-uploaded files
- Unlike M365 Copilot, a custom agent treats **user-uploaded files as essentially invisible** — you must handle them explicitly. See the dedicated [file-and-attachment-handling.md](../reference/file-and-attachment-handling.md).

---

### Sources (secondary / practitioner)
- Praneet S — *Copilot Studio Deep Dive* — https://medium.com/@praneetsy/copilot-studio-deep-dive-a2a086e22f73
- Khushboo Nijhawan — *Building Agents in Copilot Studio: the guide I wish existed* — https://medium.com/@khushboo.nijhawan/building-agents-in-copilot-studio-heres-the-guide-i-wish-existed-when-i-started-f31c33bc640a
- Mohit Aggarwal — *What I learned about Topics* — https://medium.com/product-powerhouse/i-built-a-copilot-studio-agent-that-actually-worked-heres-what-i-learned-about-topics-1b521d310fee
- peafowlit — *Multi-Agent Orchestration: a practical architecture guide* — https://medium.com/@peafowlit/multi-agent-orchestration-with-copilot-studio-a-practical-architecture-guide-f655597371b1
- George Karapetyan — *6 practical ways to handle user-uploaded files* — https://medium.com/@georgekar91/6-practical-ways-to-handle-user-uploaded-files-in-copilot-studio-a729f06642be
- *Building Autonomous AI Agents: a deep dive into Copilot Studio's full experience* — https://medium.com/codex/building-autonomous-ai-agents-a-deep-dive-into-copilot-studios-full-experience-687b553ea7a8
- Matthew Devaney — *How to build a Copilot Studio agent with generative orchestration* — https://www.matthewdevaney.com/how-to-build-a-copilot-studio-agent-with-generative-orchestration/
