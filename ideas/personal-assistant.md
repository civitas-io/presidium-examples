# Idea: Personal AI Assistant

**Domain:** Consumer  
**Complexity:** Medium–High  
**Status:** Backlog — pending design discussion

---

## The Concept

The enterprise examples show Presidium governing agents that affect organizations. This example shows the same primitives governing an agent that affects *you* — your calendar, your email, your files, your home.

This is the "regular folks" version. The governance story is identical; the stakes are personal rather than organizational.

---

## The Problem

"The assistant reorganized my email archive. I can't find anything. It also accepted a meeting invite I would have declined."

Personal AI assistants fail the same way enterprise agents do: no declared intent, no approval gates on irreversible actions, no audit of what happened and why. The difference is that the person who gets hurt is the user, not a company.

---

## What Makes This Different from Enterprise Examples

- No external IdP, no OAuth enterprise flows — the user *is* the authority
- Consent model is personal and conversational, not policy-file-based
- Reversibility matters more than in enterprise (you can reopen a ticket; you can't unsend an email)
- Trust feedback is immediate and direct — user says "that was wrong," agent records it

---

## Agent Capabilities (Governed)

| Capability | Default | Approval required |
|---|---|---|
| Read email / calendar | ALLOW | — |
| Draft email | ALLOW | — |
| Send email | REQUIRE_APPROVAL | Always |
| Accept / decline calendar invites | REQUIRE_APPROVAL | Always |
| Create calendar events | ALLOW for personal, REQUIRE_APPROVAL for inviting others |
| Move / label email | ALLOW | — |
| Delete email | REQUIRE_APPROVAL | Always |
| File access (read) | ALLOW within declared scope | — |
| File modification / deletion | REQUIRE_APPROVAL | Always |
| Home automation (lights, locks) | Configurable per device | Locks: always |

---

## Governance Story

| Primitive | How it appears |
|---|---|
| **Personal approval gates** | Send email, accept invite, delete anything → always asks first. Not optional. |
| **Intent declaration** | "I'm going to process this week's unread email" — if assistant pivots to a different task mid-session, user is notified |
| **Reversibility-first** | Agent prefers reversible actions (archive vs delete, draft vs send) unless user explicitly overrides |
| **Personal audit trail** | Full log of what the agent did, what it accessed, what it sent — user-owned, exportable |
| **Trust feedback** | User corrects the agent → trust score for that action class decays → agent asks more often in that domain |
| **Context budget** | Long-running sessions (e.g. processing a full inbox) stay within budget; agent summarizes and checkpoints rather than accumulating |

---

## Why It Matters for Presidium

Enterprise buyers want to see governance at the enterprise level. But *individuals* making decisions about whether to trust an AI assistant are the same people who make purchasing decisions at work. A personal assistant that demonstrably respects boundaries — and shows you exactly what it did and why — builds the trust that enterprise sales depend on.

It's also the clearest possible demonstration that the governance model is general, not just for compliance teams.

---

## Design Questions (Open)

- Should this be framed around a specific integration (Gmail + Google Calendar + Home Assistant)?
- Is this the "OpenClaw/Hermes" concept from design discussions? Needs clarification before scoping.
- Homelab integration (Lumi, vigil) — is this the internal dogfood version?
