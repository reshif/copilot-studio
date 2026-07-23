# Reference — Licensing & Copilot Credits (the economics)

Understanding cost shapes design decisions (generative vs classic, tools vs knowledge, reasoning models, autonomous frequency). **Copilot Credits** are the unit measuring agent usage; capacity is **pooled across the tenant**.

> Always re-verify against the official **Microsoft Copilot Studio Licensing Guide** and **Copilot Credit Guide** — pricing changes. Use the **Copilot Studio agent usage estimator** (microsoft.github.io/copilot-studio-estimator) to forecast.

## Purchase models
- **Prepaid message/credit packs** — ~**$200/pack/month for 25,000 Copilot Credits**.
- **Pay-as-you-go** — ~**$0.01 per Copilot Credit**, billed monthly to an Azure subscription, no upfront commit; **overage enforcement doesn't apply** (overage bills to Azure).
- **Pre-purchase commit** — prepaid Commit Units chosen upfront.
- **Per-user** subscription options also exist (predictable access).
- **BYO models** (Azure Foundry) are billed **separately**, not by these rates.

## Billing rates (per feature)
| Feature | Credits | M365 Copilot-licensed user |
|---------|---------|----------------------------|
| Classic answer | 1 | No charge |
| Generative answer | 2 | No charge¹ |
| Agent action (trigger, deep reasoning, topic transition; **CUA**) | 5 | No charge |
| Tenant graph grounding (per message) | 10 | No charge |
| Agent flow actions (per 100 actions) | 13 | No charge² |
| AI tools — basic (per 10 responses) | 1 | No charge |
| AI tools — standard (per 10 responses) | 15 | No charge |
| AI tools — premium (per 10 responses) | 100 | No charge |
| Content processing tools (per page) | 8 | No charge |

Voice: Classic Voice **10**/min, GenAI Voice **35**/min, Premium GenAI Voice **75**/min (core agent activity included).

¹ Generative answers are charged unless created in **Agent Builder (M365)** without tenant graph grounding. ² Agent-flow "no charge" for M365 users applies **only** to the **"When an agent calls the flow"** trigger; other triggers bill normally. CUA is **not** included in the M365 USL.

## Reasoning-model billing (two meters)
When an agent uses a **reasoning-capable** model:
**Total = feature rate (e.g. generative answer = 2) + Text-and-generative-AI-tools (premium) at 10 credits per 1K tokens** for the reasoning tokens. The premium meter covers the extra deep-reasoning compute.

## Worked examples
- **Customer support** (4 classic + 2 generative answers/run, 900/day): `[(4×1)+(2×2)] × 900 = 7,200 credits/day`.
- **Sales (tenant-graph-grounded, M365 Chat)** (4 generative + 4 tenant-graph/run, 100 unlicensed users): `[(4×2)+(4×10)] × 100 = 4,800 credits/day`.
- **Order processing (autonomous, 4 actions)**: `4×5 = 20 credits/run`.

A single complex turn can stack meters, e.g. tenant-graph grounded generative answer = **12** (10 + 2).

## Overage enforcement (prepaid capacity)
- **Threshold: 125%** of prepaid capacity → **custom agents disabled** (ongoing conversation isn't interrupted; new invocations rejected).
- Users see "There is a billing issue." / "This agent is currently unavailable. It has reached its usage limit."
- Admin notified by email + PPAC.
- **Capacity honoring**: environment allocations are respected; unallocated environments draw from remaining tenant pool; an environment with its own allocation isn't enforced while it still has its own capacity even if the tenant is at 125%. **Pay-as-you-go environments aren't impacted** (their meter kicks in).
- Fixes: reallocate capacity, buy more, or enable pay-as-you-go.

## Agent-flow enforcement (separate)
When prepaid capacity is exhausted, **new agent-flow runs are blocked**; **in-progress finish**; the **parent agent still works** for non-flow interactions; resets monthly. **Test runs and M365-user "When an agent calls the flow" runs are exempt.** Power Automate **cloud flows** use Power Automate licensing — not subject to this. See [prompts-and-agent-flows.md](prompts-and-agent-flows.md#part-2--agent-flows).

## Managing & monitoring
- **PPAC → Licensing → Copilot Studio** → **Manage Copilot credits** (reallocate), **Environments** (per-env consumption incl. **Agent flow actions** line), **Manage Agents** (set **per-agent monthly caps** before enforcement).
- View consumption reporting in PPAC.

## Design implications (cost-aware building)
- **Classic answers (1) < generative answers (2) < agent actions (5) < tenant-graph (10)** — reserve expensive features for where they add value.
- **Reasoning models** roughly *add* premium-token cost on top — use deep reasoning selectively.
- **Autonomous agents** multiply cost by trigger frequency — scope triggers tightly.
- **Multi-agent** hops each cost (agent actions) — don't over-split.
- **M365 Copilot-licensed employee-facing** usage is largely **no charge** — leverage it for internal agents.
- Cap per-agent spend and prefer **pay-as-you-go** for spiky workloads to avoid hard 125% cutoffs.
