# Example: HR Policy Assistant

**Difficulty:** Easy  
**Status:** Planned

---

## The Problem

"We built a Slack bot that answers HR questions. Then someone asked it about a colleague's salary and it answered. We turned it off."

This is the most common first-wave enterprise AI failure: an agent with no credential scoping, no audit trail, and no access boundaries. The fix isn't a smarter prompt — it's structural governance.

---

## What This Example Shows

A Slack-integrated HR assistant that answers questions about policies, benefits, and PTO balances. The agent can only see data belonging to the employee asking — enforced at the credential level, not in the agent's code.

### Governance primitives demonstrated

| Primitive | How it appears |
|---|---|
| **OBO credential scoping** | Vault issues a token scoped to `(agent_id, user_id)` — cross-employee queries fail at the token level before reaching the HR system |
| **Grants** | Agent holds `hr:read:own` — not `hr:read:all` |
| **Audit trail** | Every query logged: `(agent_id, user_id, query, timestamp, data_accessed)` |
| **DENY on grant violation** | Attempt to retrieve another employee's record → hard DENY + alert, not silent failure |
| **Rate limiting** | HR system protected from flooding during open enrollment |

---

## Architecture

```
Slack message
    └── HR Assistant Agent (Civitas)
            ├── GovernedToolProvider (Presidium)
            │       ├── Check grants: hr:read:own
            │       └── Log action to audit trail
            └── CredentialVault
                    └── OBO token scoped to requesting user_id
```

---

## Failure Mode Demonstrated

Without governance: agent has full HR system access. One prompt-injection or misconfigured query returns another employee's compensation data.

With Presidium: the vault token physically cannot retrieve another user's record. No prompt engineering required. The boundary is cryptographic.

---

## Setup

> Implementation coming. Environment variables and run instructions will be here.
