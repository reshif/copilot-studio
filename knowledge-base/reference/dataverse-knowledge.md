# Reference — Dataverse as a Knowledge Source (tuning)

Grounding an agent in **Dataverse tables** turns your business data into answerable knowledge via a RAG technique inside Dataverse. This is the most "structured" knowledge source and the one that most rewards **tuning** (synonyms + glossary).

## Prerequisites (all required)
- **Dataverse search** enabled for the environment (admin: Configure Dataverse search; select searchable fields/filters per table). If you can't add a table, this is usually why.
- Agent auth = **Authenticate with Microsoft** (No-auth and manual auth are **not** supported).
- The maker has **READ** permission (security role) on the tables — missing permissions hide tables at setup.

## Add Dataverse knowledge
Overview/Knowledge page → **Add knowledge** → **Dataverse** → pick up to **15 tables per source** (recommendations are based on the agent name) → review **name + description** (write a detailed description — orchestration routes on it) → optionally add **synonyms** and **glossary terms** → **Add to agent**.

## Limits (recap from [publishing-security-limits.md](publishing-security-limits.md))
- **2 Dataverse sources per agent**, **15 tables per source**.
- **Standard or activity** tables (plus **virtual** tables only for the **Finance & Operations** data provider).
- Synonym name ≤100 / description ≤1,000 chars; glossary name ≤100 / description ≤1,000 chars.
- Generative mode: unlimited inputs; classic mode: 2 sources.

## Tuning — the important part
Synonyms and glossary definitions are **grounding metadata** that dramatically improve recognition and answer quality. Updates take **up to ~15 minutes** to apply.

### Synonyms — for opaque columns
When a column name or values aren't self-describing (e.g. a coded column `cr_123_abc` holding city codes), add a **description/synonym** telling the AI how to interpret it:
> "cr_123_abc represents the departure city for each flight represented by the flight code."

### Glossary — define domain terms/acronyms/rules
| Scenario | Term | Sample definition |
|----------|------|-------------------|
| Acronym | VP | "VP" refers to the **Vice President** value in the `JobTitle` column of the `Contact` table. |
| Custom ownership | activity owner | The `PartyId` column in the `ActivityParty` table identifies the "activity owner." |
| Custom field | opportunity revenue | "Opportunity revenue" refers to the `Custom Revenue` column in the `Opportunity` table. |
| Complex rule/filter | overdue task | "Overdue task" = the `task` table where `statecode` = open AND `scheduledend` is earlier than today. |

Test different definitions to see which returns the best result.

## Multiline text & File columns (preview)
Apply **unstructured reasoning** over **Multiline Text (`MemoType`)** and **File (`FileType`)** columns for higher-quality answers:
- Requires Dataverse search on + maker/admin access.
- In **Power Apps**, mark the table and those columns **Searchable**, add them to the **Quick Find View** (`Edit find table columns`), **Save and Publish**.
- Creating a search index adds **Dataverse capacity cost**.
- Caveats: if you added the Dataverse source before configuring the columns, backfill can take **up to 2 days** (re-add to expedite); non-org-language content in File attachments isn't supported.

## Guidance
- Prefer Dataverse knowledge for **structured business records** (accounts, tickets, orders) you want the agent to reason over.
- Invest in **synonyms + glossary** early — it's the highest-ROI tuning for Dataverse answers.
- Keep the **source description** specific and distinct (routing). Pair with clear instructions ("search the Opportunities table for pipeline questions").
- Watch the 2-source / 15-table caps; combine tables thoughtfully rather than adding many sources.
- Distinct from using Dataverse via a **tool** (read/write actions) — knowledge = read/ground; tool = act. See [tools.md](tools.md).
