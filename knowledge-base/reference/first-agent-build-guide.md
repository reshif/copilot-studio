# Reference — Build Your First Agent (end-to-end walkthrough)

A concrete, code-first path that exercises everything in the KB. Example: a **"Contoso Support"** agent that answers product questions from knowledge, creates a support ticket via a tool, and escalates deterministically.

## 0. Decide the design first
| Decision | Choice for this example | Why |
|----------|------------------------|-----|
| Experience | Classic (generative orchestration ON) | The VS Code extension clones classic YAML; generative orchestration gives flexible routing. |
| Orchestration | Generative | Auto-routes to knowledge/tools by description. |
| Auth | Authenticate with Microsoft | Internal users; enables tools; SSO in Teams. |
| Knowledge | 1 SharePoint source + uploaded PDF | Grounded answers. |
| Tools | `CreateSupportTicket` (agent flow) | Real action, user-context auth. |
| Topics | `greeting`, `fallback` (RAG), `escalate` (deterministic) | Blend generative + deterministic. |
| Channels | Teams | Internal rollout. |

## 1. Bootstrap
1. Portal: create a blank agent "Contoso Support", set instructions, turn **Generative AI → Orchestration = Generative**.
2. VS Code: install extension → sign in → right-click the agent → **Clone agent** → pick a Git folder → `git init`.
3. Inspect the clone (see [../04-agent-definition-yaml-schema.md](../04-agent-definition-yaml-schema.md)):
```text
contoso-support/
├── agent.mcs.yaml          settings.mcs.yml         connectionreferences.mcs.yml
├── topics/  actions/  workflows/  knowledge/files/  trigger/  icon.png
```

## 2. Instructions (agent.mcs.yaml)
Write identity/scope/limits in `instructions`. Keep ≤8,000 chars. Reference child agents with `/` if used.
```
You are Contoso Support, an internal助 assistant.
- Answer product questions ONLY from connected knowledge; always cite the source.
- To create a ticket, use the CreateSupportTicket tool; confirm details first.
- If the user is angry or asks for a human, hand off via the Escalate topic.
- If you can't ground an answer, say so and offer to create a ticket.
```
Ensure `grounding.GenerativeActionsEnabled: true` so routing uses descriptions.

## 3. Greeting topic (`topics/greeting.topic.yaml`)
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart
  id: main
  actions:
    - kind: SendActivity
      id: welcome
      activity:
        text:
          - "Hi {System.User.FirstName}, I'm Contoso Support. Ask a product question or say 'create a ticket'. (Some replies are AI-generated.)"
```

## 4. Fallback / RAG topic (`topics/fallback.topic.yaml`)
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  priority: -1
  actions:
    - kind: SearchAndSummarizeContent
      id: rag
      userInput: =System.Activity.Text
      variable: Topic.Answer
      moderationLevel: High
      additionalInstructions: Answer in <=3 sentences and always cite the source.
      sharePointSearchDataSource: {}
    - kind: ConditionGroup
      id: hasAnswer
      conditions:
        - id: found
          condition: =!IsBlank(Topic.Answer)
          actions:
            - kind: EndDialog
              id: end
              clearTopicQueue: true
      elseActions:
        - kind: SendActivity
          id: noAnswer
          activity:
            text: ["I couldn't find that in our docs. Want me to create a support ticket?"]
```
> Under generative orchestration, knowledge is searched proactively too — this topic is the explicit fallback. See [knowledge.md](knowledge.md#search-modes).

## 5. Escalate topic (deterministic) (`topics/escalate.topic.yaml`)
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Escalate to a human
    triggerQueries:
      - talk to a person
      - speak to an agent
      - this isn't helping
  actions:
    - kind: SendActivity
      id: ack
      activity:
        text: ["I'll connect you to a support representative now."]
    # Add a Transfer to agent (handoff) node here for your configured handoff product.
```
Give it a strong **description** too (generative agents route on it).

## 6. Tool: CreateSupportTicket (agent flow)
Portal (or VS Code + AI assistant) → **Tools → Add a tool → Agent flow**. Config (see [tools.md](tools.md)):
- **Details**: name `CreateSupportTicket`; description "Creates a Contoso support ticket. Use when the user reports a problem that can't be answered from knowledge. Collects product, severity, and description."
- **Inputs**: `product`, `severity`, `description` → **Dynamically fill with AI** (orchestrator asks if missing).
- **Auth**: **End-user credentials**.
- **Ask the end user before running**: **Yes** (state-changing).
- **Completion → After running**: **Send an adaptive card** confirming ticket number.
On disk: `workflows/CreateSupportTicket/metadata.yaml` + `workflow.json`; `connectionreferences.mcs.yml` updated.

## 7. Knowledge
- Add the SharePoint site as a knowledge source; write a distinct description (matters past 25 sources).
- Drop a PDF into `knowledge/files/`; it uploads on **Apply**.
- Settings: **Allow ungrounded responses = Off**, **Moderation = High**, add "always cite" to instructions. Enable **Semantic search** if you have M365 Copilot licensing.

## 8. Validate → Apply → Test
1. **Validate**: no Problems-pane errors; every node `id` unique; Power Fx starts with `=`; no `TODO`/`_REPLACE`; kinds exist in schema. (Use the `skills-for-copilot-studio` `validate` skill if adopted — see [../11-code-first-framework-and-workflow.md](../11-code-first-framework-and-workflow.md).)
2. **Commit** to Git.
3. **Preview/Get** remote changes (Apply is blocked otherwise), then **Apply** to the dev environment. (Apply is live-but-not-published.)
4. **Test** in the Copilot Studio test pane; open the **activity map** to confirm routing:
   - "What's the warranty on X?" → RAG/knowledge, cited.
   - "Create a ticket, my printer is broken" → asks for product/severity, confirms, calls tool.
   - "talk to a person" → Escalate topic.
5. **Evaluate**: build a test set (new-experience Evaluate tab) for regressions.

## 9. Ship
- Package in a **solution**; promote dev→test→prod via **Power Platform Pipelines** (reconnect connection references, set env variables per env). See [../10-alm-publishing-testing.md](../10-alm-publishing-testing.md).
- **Publish** in prod; add the **Teams** channel; publish-for-yourself first, then make available to others.
- **Monitor** post-deploy (Monitor tab / analytics).

## 10. Pre-flight checklist
- [ ] Instructions: identity, scope, cite-sources, tool/escalation rules (≤8k chars).
- [ ] Generative orchestration ON; every topic/tool/knowledge/agent has a **distinct, specific description**.
- [ ] Routable choices (tools+topics+agents) ≤ ~30; tools ≤ 25–30.
- [ ] Deterministic flows (escalation, ticket confirm) authored as topics/cards, not left to the model.
- [ ] Auth mode set; `User.AccessToken` never in Message nodes/untrusted flows; state-changing tools use **Ask before running**.
- [ ] Knowledge: descriptions distinct; ungrounded off; moderation set; "always cite" in instructions.
- [ ] No `.` in topic names (breaks solution export); unique node IDs; `=`-prefixed Power Fx.
- [ ] Validated → committed → Preview/Get → Apply to **dev** (never straight to prod) → tested via activity map → evaluated.
- [ ] Promotion via solution/pipeline; **Publish** only after prod validation.

## Common pitfalls (learned from the docs)
- **Apply ≠ Publish** — Apply updates the definition for testing; Publish exposes to users. Re-publish after every change; latest content only reaches users on a new session (or `start over`).
- Generative orchestration **ignores** Conversational-boosting customizations and **doesn't** call Multiple-Topics-Matched — turn that system topic off while testing.
- Custom entities **can't** be tool/topic **input parameters** — collect via a Question node.
- Connection references & env variables are **per-environment** — solutions/pipelines rebind them; ad-hoc reattach doesn't.
- No-auth agents **can't** run credentialed tools.
