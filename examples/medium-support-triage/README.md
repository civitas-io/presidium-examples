# Example: Customer Support Triage + Draft

**Difficulty:** Medium  
**Status:** Planned

---

## The Problem

"We automated first-response drafting. The agent auto-sent 200 emails with the wrong pricing. Customer relations disaster."

The agent had ALLOW on both `read_ticket` and `send_email`. Every individual action was within policy. The sequence — read → classify → send without review — was not.

---

## What This Example Shows

An agent that ingests support tickets, classifies severity, pulls customer history, and drafts responses. It can send low-stakes responses autonomously. High-stakes or off-plan actions require human approval.

### Governance primitives demonstrated

| Primitive | How it appears |
|---|---|
| **Intent declaration** | Agent declares at task start: "I will read ticket, look up purchase history, draft response." Attempting to query billing or payment data triggers DriftPolicy |
| **REQUIRE_APPROVAL gate** | Any outbound email above a configurable stakes threshold → human approval required before send |
| **Drift detection** | Agent deviates from declared plan → DriftPolicy: WARN / BLOCK / REQUIRE_REDECLARATION |
| **Context budget** | Complex 50-message threads get a token cap; agent summarizes rather than accumulating unbounded context |
| **Trust score feedback** | Drafted responses repeatedly rejected by support staff → trust decays → more drafts routed to approval |

---

## Architecture

```
Ticket ingestion
    └── Support Triage Agent (Civitas)
            ├── IntentDeclaration → PolicyEngine
            ├── GovernedToolProvider
            │       ├── read_ticket → ALLOW
            │       ├── lookup_customer_history → ALLOW (in plan)
            │       ├── query_billing → BLOCK (not in declared plan)
            │       └── send_email → REQUIRE_APPROVAL (above threshold)
            ├── ContextWindow (per-session token budget)
            └── TrustScore (feedback from approval outcomes)
```

---

## The Trust Feedback Loop

This example runs best over multiple tickets. After 10–20 rounds:
- Drafts that get approved with no edits → trust score increases
- Drafts that get rejected or heavily edited → trust score decays
- Below threshold: more drafts require approval automatically

This is the "earn autonomy" mechanic in a medium-complexity setting. The agent starts supervised and relaxes toward autonomous operation as it demonstrates accuracy.

---

## Failure Mode Demonstrated

Without governance: agent with ALLOW on `send_email` eventually sends the wrong thing. No audit of what it sent or why. No mechanism to tighten supervision after a bad run.

With Presidium: the approval gate prevents the bad send; the trust feedback loop means a bad run automatically increases scrutiny on subsequent runs.

---

## Setup

> Implementation coming. Environment variables and run instructions will be here.
