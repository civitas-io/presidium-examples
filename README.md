# Presidium Examples

> Real-world governed agent workflows built with [Civitas](https://github.com/civitas-io/python-civitas) + [Presidium](https://github.com/civitas-io/presidium).

Enterprises have invested heavily in "going agentic" — and have little to show for it. The failure mode is always the same: agents that touched something they shouldn't, costs that ran away, actions nobody can audit, systems nobody trusts. These examples recreate those exact scenarios and show them handled correctly.

Each example is self-contained: clone it, set your env vars, run it.

---

## Examples

| Example | Difficulty | Governance primitives demonstrated |
|---|---|---|
| [HR Policy Assistant](examples/easy-hr-assistant/) | Easy | Identity · Credential vault (OBO) · Grants · Audit trail |
| [Customer Support Triage](examples/medium-support-triage/) | Medium | Intent declaration · Approval gates · Trust feedback · Context budget |
| [SOC Incident Response](examples/hard-soc-automation/) | Hard | Trust-gated autonomy · Earn-autonomy arc · Multi-agent supervision · Drift detection |

---

## Governance Primitive Index

If you're looking for a specific Presidium capability, start here:

| Primitive | Where to see it |
|---|---|
| Credential vault + OBO token scoping | HR Assistant |
| Per-action audit log | HR Assistant, all examples |
| Intent declaration | Support Triage, SOC |
| REQUIRE_APPROVAL gate | Support Triage, SOC |
| Trust score decay + feedback | Support Triage, SOC |
| Context budget enforcement | Support Triage, SOC |
| Trust-gated autonomy (observe → advise → act) | SOC |
| Multi-agent supervision tree | SOC |
| Drift detection (goal hijacking) | SOC |

---

## Ideas: More Complex Workflows

See [ideas/](ideas/) for detailed briefs on workflows planned for future examples. These cover harder coordination problems, longer-running workflows, and higher-stakes domains.

---

## Requirements

```
python >= 3.12
civitas >= 0.x
presidium >= 0.x
```

Each example has its own `README.md` with setup instructions and required environment variables.

---

## Philosophy

These examples are deliberately not "happy path" demos. Each one is designed around a known real-world AI failure — and shows exactly what went wrong, and how Presidium's governance layer handles it. The goal is to give enterprise teams confidence in the specific places they expect agents to fail.
