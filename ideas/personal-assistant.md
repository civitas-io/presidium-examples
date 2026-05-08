# Idea: Personal AI Assistant (Jarvis)

**Domain:** Consumer  
**Complexity:** Medium–High  
**Status:** Backlog — pending scope decision

---

## The Concept

Decision fatigue is the hidden tax on modern life. Hundreds of micro-decisions daily — which emails need a reply, which meeting invite to accept, which bill is due, whether to reschedule that dentist appointment — each small, but collectively exhausting. Most of them don't need you. They just need judgment.

Jarvis handles the 80% autonomously. It surfaces the 20% that actually need you — prepared, contextualized, and with a draft action ready to approve or redirect.

---

## Life Domains Covered

| Domain | What Jarvis does autonomously | What it surfaces to you |
|---|---|---|
| **Email** | Labels, archives, flags action items, drafts replies | Replies ready to send, anything requiring a decision |
| **Calendar** | Detects conflicts, preps meeting briefs, drafts declines | Invite acceptance, schedule changes that affect others |
| **Messages** | Drafts replies for low-stakes threads | Anything emotionally significant, anything requiring a commitment |
| **Bills & Finance** | Tracks due dates, flags anomalies, reconciles against budget | Payments above threshold, unusual charges, budget overruns |
| **Health** | Tracks appointments, preps reminders, flags gaps in commitments | Rescheduling, anything requiring a decision about your care |
| **Social** | Surfaces birthdays, anniversaries, follow-ups you've let drift | Draft messages, nothing sent without review |

---

## The Governance Story

The failure modes for a personal assistant are not corporate failures — they're personal ones. The assistant sent an email you wouldn't have sent. It accepted a commitment you would have declined. It deleted something you needed. These are violations of autonomy, not policy.

Presidium's governance primitives map to this naturally:

| Primitive | How it appears for Jarvis |
|---|---|
| **Irreversibility gate** | Send email, accept invite, pay bill, delete anything → always requires approval. Non-negotiable. |
| **Reversibility-first** | Agent prefers reversible actions: archive over delete, draft over send, suggest over book |
| **Intent declaration** | "Processing today's inbox" — if Jarvis drifts to touching calendar or finances mid-session without being asked, you're notified |
| **Approval with context** | Every approval request includes: what the action is, why Jarvis thinks it's right, what happens if you decline |
| **Trust feedback** | "Don't do that" → trust decays for that action class in that context → Jarvis asks more often there. Corrections persist. |
| **Personal audit trail** | Full log of every action, every draft, every decision Jarvis made on your behalf — yours, local, exportable |
| **Context budget** | Long inbox sessions don't spiral into 100k-token context swamps; Jarvis checkpoints and summarizes rather than accumulating everything |

---

## What Makes This Different from Existing Assistants

Siri, Google Assistant, Alexa: reactive. They answer questions and execute explicit commands.

Jarvis: proactive, but governed. It acts on your behalf without being asked — and the governance layer is what makes that safe to do. You don't have to trust that Jarvis will make the right call every time. You trust that it will never do something irreversible without checking, and that it will show you exactly what it did.

The transparency is the product. The audit trail isn't a compliance feature — it's the thing that lets you delegate confidently.

---

## Why It's the Right Consumer Example

Enterprise buyers are also humans with decision fatigue. Showing Presidium's governance model applied to something as personal and relatable as "I got 200 emails today and Jarvis prepared 40 draft replies for me to approve" makes the value proposition immediate and human-scale.

It's also the bridge between the enterprise demos and everyday life: the same trust model, the same approval gates, the same audit trail — but for your inbox, not your SOC.

---

## Connection to Existing Work

This may be the evolved form of the M1.8 hero demo (personal AI assistant with homelab integrations). If so, Jarvis is the governed version of that — same integrations, Presidium governance layer added.

Integrations to scope:
- Gmail + Google Calendar (MCP, already explored in M3)
- iMessage / WhatsApp (platform-dependent)
- Banking / Plaid (financial feeds)
- Apple Health / Garmin (health data)
- Home automation (Home Assistant / homelab)

---

## Open Questions

- Does this become a standalone product (consumer-facing) or a showcase example only?
- Local-first vs cloud — personal data sensitivity argues for local-first deployment
- Should the approval UX be a mobile app, a Slack bot, or something else?
