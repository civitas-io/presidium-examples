# Idea: Supply Chain Risk + Emergency Procurement

**Domain:** Operations / Procurement  
**Complexity:** High  
**Status:** Backlog

---

## The Problem

"There was a port disruption. The agent placed emergency orders with 3 alternative suppliers at spot prices, totaling $4M in commitments, without anyone signing off."

Supply chain automation is operationally valuable and financially dangerous. The failure mode is not malice — it's an agent correctly following its goal (restore supply continuity) without the constraints that make that goal safe to pursue autonomously.

---

## Agent Pipeline

```
GovernedSupervisor
    ├── Risk Monitor Agent     — watches supplier feeds, port status, lead times
    ├── Sourcing Agent         — identifies alternative suppliers, gets quotes
    ├── Negotiation Agent      — proposes terms within pre-approved parameters
    ├── PO Draft Agent         — creates purchase orders for review
    └── ERP Submission Agent   — posts approved POs to procurement system
```

---

## Governance Story

| Primitive | How it appears |
|---|---|
| **Intent declaration** | Agent declares "I will identify alternatives and draft POs up to $50K" — submitting above that triggers DriftPolicy BLOCK |
| **Spend-ceiling credentials** | ERP OAuth token has a pre-approved spend ceiling baked into the token scope — agent physically cannot commit above the ceiling |
| **REQUIRE_APPROVAL** | Any PO above threshold → procurement manager approval required |
| **Trust decay** | Agent's sourcing recommendations repeatedly leading to late delivery → trust decays → more sourcing decisions require human sign-off |
| **Audit trail** | Every supplier contact, every quote, every commitment — logged with the disruption event that triggered it |

---

## Why Spend-Ceiling Credentials Matter

Most governance systems check spend limits in policy. Presidium enforces them at the credential level: the agent's ERP token is issued with a spend ceiling as a claim. Even if the policy check is bypassed or the agent reasons its way around it, the token physically cannot authorize a transaction above the ceiling. This is the difference between a guardrail and a wall.

---

## Why It's Compelling

Post-COVID, every supply chain team understands disruption risk. The procurement automation dream is real. The "$4M committed without approval" failure is also real, and happened at multiple large manufacturers during 2021–2022.
