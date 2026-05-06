# Example: SOC Incident Response Automation

**Difficulty:** Hard  
**Status:** Planned

---

## The Problem

"We gave the agent the ability to quarantine hosts. It blocked 3 production servers during peak traffic because it misidentified a port scan from our own monitoring tool."

The agent had correct permissions and followed its policy rules. It was wrong. And it acted autonomously at machine speed before any human could intervene.

The fix is not a better model. It's an autonomy model that starts conservative and expands as accuracy is demonstrated.

---

## What This Example Shows

A multi-agent SOC pipeline that starts in **observe mode** and earns its way toward autonomous containment. Trust score is the mechanism — not a flag the operator sets, but a property the agent builds through demonstrated accuracy.

### Governance primitives demonstrated

| Primitive | How it appears |
|---|---|
| **Trust-gated autonomy** | Trust > 0.85: autonomous containment. 0.6–0.85: analyst approval required. < 0.6: advisory only, human acts |
| **Earn-autonomy arc** | Accurate triage → trust grows → approval threshold relaxes. False positives → trust decays → supervision tightens |
| **Multi-agent supervision tree** | Five agents under a GovernedSupervisor; crash in one doesn't corrupt others' state |
| **Credential scoping** | Containment agent can quarantine (network isolation); vault blocks host deletion — not a policy check, a physical token boundary |
| **Intent declaration** | Investigation agent declares which alert it is pursuing; pivoting to a different alert mid-investigation triggers DriftPolicy |
| **Context budget** | Complex multi-vector incidents don't spiral; investigation agent summarizes rather than accumulating unbounded timeline |
| **Audit trail** | Every containment action, every approval, every false positive logged with the human who reviewed it |

---

## Architecture

```
Alert stream (SIEM)
    └── GovernedSupervisor (Presidium)
            ├── Triage Agent
            │       └── classifies alert severity, routes to investigation
            ├── Investigation Agent
            │       ├── IntentDeclaration (this alert, these hosts)
            │       ├── ContextWindow (token budget per incident)
            │       └── pulls logs, correlates events, proposes verdict
            ├── Containment Agent
            │       ├── TrustScore check → determines approval requirement
            │       ├── GovernedToolProvider: quarantine_host → trust-gated
            │       └── CredentialVault: no delete_host token issued
            ├── Remediation Agent
            │       └── REQUIRE_APPROVAL for any persistence changes
            └── Post-Incident Agent
                    └── generates report, updates trust scores from outcome
```

---

## The Autonomy Arc

This example is designed to be run across a sequence of simulated incidents. The arc:

1. **Day 1:** All containment actions require analyst approval (trust starts at baseline)
2. **After 20 accurate triages:** Trust crosses 0.85 → low-severity containment becomes autonomous
3. **One false positive:** Trust decays → approval threshold drops back → operator sees it happen in real time
4. **Recovery:** Accurate run of 10 → trust rebuilds

This is the governance story: not a static policy, but a living trust relationship between the agent and the operators who depend on it.

---

## Failure Mode Demonstrated

Without governance: accurate at 95% — which means 1 in 20 containment actions is wrong. At machine speed across a real SOC alert volume, that's multiple false positives per day, each potentially taking down production infrastructure.

With Presidium: the 95% agent operates autonomously only after demonstrating that accuracy. The 5% failure rate is caught by the approval gate during the trust-building phase. Operators see the full audit of what the agent did and why, and can tune thresholds to match their risk tolerance.

---

## Setup

> Implementation coming. Environment variables and run instructions will be here.
