# Idea: Clinical Data Pipeline (Healthcare / Pharma)

**Domain:** Healthcare  
**Complexity:** Very High  
**Status:** Backlog

---

## The Problem

"The adverse event detection agent queried across patient cohorts without checking consent scope. HIPAA audit found it. $2M fine."

Healthcare is one of the largest enterprise AI spenders. The governance requirements are not aspirational — they are legally mandated (HIPAA, FDA 21 CFR Part 11, EU MDR). Failure is not a performance issue; it's a regulatory event.

---

## Agent Pipeline

```
GovernedSupervisor
    ├── Ingestion Agent         — pulls EHR data within consent scope
    ├── Cohort Builder          — assembles patient cohorts for trial arm analysis
    ├── Adverse Event Detector  — flags potential adverse events across cohort
    ├── Signal Validator        — assesses signal strength, eliminates confounders
    ├── Report Generator        — drafts regulatory submission (FDA eMDR / EMA)
    └── Submission Agent        — submits to regulatory system with named attestor
```

---

## Governance Story

| Primitive | How it appears |
|---|---|
| **Consent-scoped credentials** | Vault token is issued only for patients who consented to research use — cross-cohort queries fail at the token level, not a policy check |
| **REQUIRE_APPROVAL + named attestor** | Regulatory submission requires a named human (clinical PI or regulatory affairs lead) to attest before the agent submits to FDA |
| **Audit trail as regulatory artifact** | Every agent action, every data access, every flagged event — the audit log *is* the 21 CFR Part 11 audit trail that regulators will request |
| **Multi-agent supervision** | Signal validator crash during analysis doesn't corrupt adverse event flags; supervisor preserves state independently |
| **Context budget** | Longitudinal patient records spanning years don't blow context; agents process in windowed epochs |

---

## Signature Primitive: Consent-Scoped Vault Tokens

Patient consent is not a filter applied after data retrieval. The vault token is issued only for consented patients — the agent cannot retrieve records for non-consenting patients because the token claim is mathematically scoped. This is the architecture that makes HIPAA compliance structural rather than procedural.

---

## Why It's the Hardest Example

Requires: EHR integration (FHIR API), realistic synthetic patient data, FDA eMDR format knowledge, and a regulatory workflow that spans weeks in production. Best demonstrated against a synthetic dataset with a realistic FHIR facade.

---

## Why It's Worth Building

Healthcare AI spend is enormous. HIPAA compliance is non-negotiable. Every health system CTO building AI workflows needs exactly this architecture. A working, governed clinical pipeline example is a direct sales artifact for that market.
