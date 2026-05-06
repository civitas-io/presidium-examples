# Idea: Legal Contract Review

**Domain:** Legal  
**Complexity:** High  
**Status:** Backlog

---

## The Problem

"We ran 200 NDAs through the agent. It flagged all of them as low-risk. Three had uncapped liability clauses. One became a $12M dispute."

Every GC is being asked to automate contract review. Every GC is also afraid of exactly this failure. The question is not whether the agent is accurate enough — it's whether the governance layer degrades gracefully when it isn't.

---

## Agent Pipeline

```
GovernedSupervisor
    ├── Ingestion Agent        — parses PDF/DOCX, segments into clauses
    ├── Extraction Agent       — identifies clause types, parties, obligations
    ├── Risk Classification    — rates each clause against standard playbook
    ├── Precedent Agent        — compares to prior signed agreements
    └── Summary Agent          — produces attorney-review packet
```

---

## Governance Story

| Primitive | How it appears |
|---|---|
| **Outcome-driven trust decay** | Contract rated low-risk → later generates a dispute → signal fed back → risk agent's trust decays → more contracts routed to attorney review |
| **REQUIRE_APPROVAL** | Any contract above $X total value or flagged as "non-standard" routes to mandatory attorney review before countersign |
| **Context budget** | 400-page acquisition agreement doesn't blow context; extraction agent processes in windowed chunks |
| **Audit trail** | Every clause rating, every risk flag, every attorney decision — logged for malpractice defense |

---

## Signature Primitive: Outcome-Driven Trust Decay

This is the primitive that no other governance framework has. Most systems evaluate trust at runtime — did the agent follow its policy? Presidium can close the loop: if a downstream outcome (dispute, loss) correlates with a prior agent decision (low-risk rating), that signal feeds back into trust.

The agent learns to be more conservative not because it was told to, but because the evidence demands it.

---

## Why It's Hard to Demo

The feedback loop is inherently slow — months between contract signing and dispute resolution. Best demonstrated with a synthetic dataset of pre-labeled contracts where outcomes are known, compressed into a single session.
