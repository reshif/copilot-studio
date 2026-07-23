# Reference — Adaptive Cards

Adaptive Cards add rich, interactive UI (text, graphics, inputs, buttons) to agents. Platform-agnostic JSON. In a cloned agent, card templates live in `adaptive-cards/*.json`.

## Two ways to use cards
- **Interactive** (collect input) → **Adaptive Card node** ("Ask with Adaptive Card"). Must contain **≥1 submit button**. User input is stored in **output variables**.
- **Informational** (display only) → add the card to a **Message** or **Question** node instead (no submit button).

## Schema versions (host-dependent!)
Copilot Studio supports **Adaptive Cards schema ≤ 1.6**, but the target host limits you:
- **Bot Framework Web Chat** (default website) — 1.6, but **no `Action.Execute`**.
- **Omnichannel live chat widget** — max **1.5**.
- **Microsoft Teams** — max **1.5**.
- Copilot Studio renders **1.6 cards only in the test chat**, not on the canvas.

Design in the **built-in Adaptive Card designer**, paste a **JSON payload**, or switch to a **Power Fx formula** for dynamic data.

## Authoring flow
1. Add node → **Ask with Adaptive Card** → Properties → **Edit adaptive card**.
2. Add elements or paste JSON in the **Card payload editor**. Include ≥1 `Action.Submit`.
3. Save — Copilot Studio auto-creates **output variables from the card's inputs** (input `id` → variable). Fix via **Edit Schema** if wrong.

## Example card JSON (input + validation)
```json
{
  "$schema": "http://adaptivecards.microsoft.com/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.5",
  "body": [
    { "type": "TextBlock", "text": "Tell us about yourself", "weight": "Bolder", "size": "Medium", "wrap": true, "style": "heading" },
    { "type": "Input.Text", "id": "myName", "label": "Your name (Last, First)",
      "isRequired": true, "regex": "^[A-Z][a-z]+, [A-Z][a-z]+$",
      "errorMessage": "Please enter your name in the specified format" }
  ],
  "actions": [ { "type": "Action.Submit", "title": "Submit" } ]
}
```
The `Input.Text` `id: "myName"` becomes an output variable holding the user's entry.

## Dynamic cards with Power Fx
Switch the node to **Formula** to bind variables (e.g. `text: Topic.Title`). The JSON is auto-converted to a Power Fx record. **One-way:** once you edit in the formula panel you can't return to JSON — **save the original JSON in a comment/notes first**.

## Reprompt / interruption behavior
If the agent awaits a card submit but the user types text (and it doesn't trigger an interruption):
- **How many reprompts** — Repeat up to 2 (default) / Repeat once / Don't repeat (resends the card each retry).
- **Retry prompt** — custom message on retry (**Customize**).
- **Allow switching to another topic** (default on) — a user message interrupts and switches topics; the card is resent after the interrupting topic ends.

## Consecutive-card pitfalls (important)
Cards allow multiple submits; with consecutive cards a user can click an **older** card's button → wrong behavior. Mitigate:
- Put a **unique identifier in each `Action.Submit` data** and validate it:
  ```json
  { "type": "Action.Submit", "title": "Confirm",
    "data": { "actionSubmitId": "booking_confirm_card_v3_confirm" } }
  ```
- In **custom web chat**, disable a card's buttons after first click (guard against stale/duplicate submits) — re-render/mutate the DOM and de-dupe on the `actionSubmitId`.

## Guidance
- Use cards for **confirmations, structured input, and rich display** where channels support them (mind the 1.5/1.6 limits per channel — see [publishing-security-limits.md](publishing-security-limits.md)).
- Remember **generative-answer customization** via cards means you must render **citations yourself** (see [knowledge.md](knowledge.md)).
- Instructions **can't** change when a card triggers — edit the card/its trigger phrases.
