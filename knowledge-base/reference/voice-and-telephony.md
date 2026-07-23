# Reference — Voice & Telephony (IVR)

Copilot Studio agents can run as **interactive voice response (IVR)** agents over the **Telephony** channel (via **Azure Communication Services** phone numbers), integrated with **Dynamics 365 Contact Center**.

## Two kinds of voice agent
| Type | Orchestration | Engine | Use for |
|------|---------------|--------|---------|
| **Basic voice agent** | Classic | NLU (intent/entity) + voice optimization | Controlled, topic-driven IVR (menus, DTMF) |
| **Real-time voice agent** | Generative | Speech-to-speech (STS) real-time model | Low-latency, natural conversation |

Voice agents support **speech + DTMF** input, context variables, and **call transfer**.

## Enable voice
Settings → **Voice** → **Enable voice** → choose **Basic** (classic/topics) or **Real-time voice** (generative). This is a **modality switch** (Text vs **Speech & DTMF** — mutually exclusive), NOT the channel — you must also turn on the **Telephony** channel. Enabling voice adds system topics: **Silence detection**, **Speech unrecognized**, **Unknown dial pad press**, plus **Answering Machine Detection** and (for new voice agents) a **Main Menu** topic with DTMF triggers. Changes take effect only after **Publish**.

> If **Enable voice** is greyed out, turn off "Get new features early" for the environment. **Increase accuracy with agent data** only helps agents using **trigger phrases** (not topic descriptions). Turning voice **off** with Telephony enabled can **break** the agent (disables all DTMF triggers).

## Agent-level voice settings (timeouts) — with defaults
| Group | Setting | Default | Meaning |
|-------|---------|---------|---------|
| DTMF | Interdigit timeout | 3,000 ms | Max wait for next key (multi-digit). |
| DTMF | Termination timeout | 2,000 ms | Wait for a terminating key at max length. |
| Silence detection | Silence detection timeout | **No timeout** | Max silence waiting for input (default = wait forever). |
| Speech collection | Utterance end timeout | 1,500 ms (max 3,000) | Pause length that means "done speaking." |
| Speech collection | Speech recognition timeout | 12,000 ms | Time allowed once user starts speaking. |
| Latency messaging | Send message delay | 500 ms | Wait before playing the latency message. |
| Latency messaging | Minimum playback time | 5,000 ms | Min time the latency message plays. |
| Speech sensitivity | Sensitivity | 0.5 | Speech-vs-noise balance (lower for noisy environments). |

Most have **node-level overrides** (Question node → Voice properties).

## Key voice features
- **Barge-in** — let callers interrupt the agent (DTMF or speech). Set per Message/Question node (Properties → **Allow barge-in**). Turn **off** for compliance/first/updated messages. A "batch" = consecutive Message nodes ending at a Question node; barge-in resets per batch.
- **Silence detection** (per Question node): timeout (agent setting / disable / custom ms), reprompts (up to 2 / once / none), and **Fallback action**: **Hang up** (default), **Escalate**, **Set variable to value**, **Set variable to empty**, **Redirect to a topic**.
- **Latency messages** — for long backend/flow ops: audio loops until done (chat sends once). Configure on the Action node.
- **SSML** — shape TTS with tags available in the SSML menu: `<audio src>` (encode URL vars with `EncodeHTML`), `<break>`, `<emphasis>`, `<prosody>` (pitch/rate/volume), `<lang xml:lang>`. Manual tags also allowed.
- **Answering Machine Detection** system topic — leave a voicemail message.
- **Call termination** — Topic management → **End conversation** hangs up.
- **DTMF-only mode** — `System.Conversation.OnlyAllowDTMF = true` ignores speech; prefer **per-question DTMF mapping** over global (informational messages during DTMF-only can cause empty-grammar telephony errors).

## Call transfer
Topic management → **Transfer conversation**:
- **External phone number** — blind transfer to a **PSTN** or **direct routing** number (include `+` and country code). Optional **SIP UUI header** (`key=value;…`, ≤256 chars, only letters/numbers/`=`/`;`; **no variables**; requires **direct routing** — PSTN doesn't support UUI).
- **To a representative** — via **Escalate**/handoff (explicit triggers).
- Also **SIP X-headers** for advanced transfer context.

## Voice variables (transfer context to D365 Contact Center)
`System.Activity.From.Name` (caller ID), `System.Activity.Recipient.Name` (called number), `System.Conversation.SipUuiHeaderValue`, `System.Activity.UserInputType` (DTMF/speech/text), `System.Activity.InputDTMFKey`, `System.Conversation.OnlyAllowDTMF`, `System.Activity.SpeechRecognition.Confidence` (0–1), `System.Activity.SpeechRecognition.MinimalFormattedText`. (Full list in [variables-and-entities.md](variables-and-entities.md).)

## Billing (from [licensing-and-credits.md](licensing-and-credits.md))
Classic Voice **10 credits/min**, GenAI Voice **35/min**, Premium GenAI Voice **75/min** (core agent activity included).

## Guidance
- **Basic (classic)** for deterministic menu/DTMF IVR; **Real-time** for natural back-and-forth.
- Use **open-list/dynamic-inline entities** (see [variables-and-entities.md](variables-and-entities.md)) for runtime values (accounts, payees) — common in classic voice.
- Tune **silence/utterance timeouts** and **sensitivity** to the environment; keep barge-in on except for compliance/first messages.
- Design short prompts; publish is slower with large trigger-phrase/entity sets.
