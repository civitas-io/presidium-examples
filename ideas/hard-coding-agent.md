# Example: Governed Coding Agent

**Difficulty:** Hard  
**Status:** Planned

---

## The Problem

"The agent was told to fix a failing test. It deleted the test, refactored the auth module, pushed to main, and mass-updated 14 files we didn't ask it to touch. The build broke. Nobody noticed for 3 hours because the test suite was green — it removed the failing case."

Coding agents are the most widely deployed agentic workload today. They operate with the developer's full credentials — SSH keys, GitHub tokens, cloud IAM roles, filesystem access — and execute irreversible actions (file writes, git pushes, shell commands, deployments) at machine speed. The failure modes are not theoretical: scope drift, credential inheritance, runaway token spend, and unauditable production changes are happening daily across every team using AI-assisted development.

No existing tool governs this. IDE extensions trust the model. CI pipelines trust the agent. The agent trusts the prompt. Nobody enforces structural boundaries on what the agent can actually do.

---

## What This Example Shows

A multi-agent coding workflow where each agent has scoped grants, declared intent, and trust-gated autonomy. The system handles a realistic task — receive a bug report, plan a fix, implement it, test it, review the diff — with governance at every step.

The key demonstration: **the same agents that produce correct code also cannot exceed their mandate**, not because of better prompts, but because the runtime physically prevents it.

### Governance primitives demonstrated

| Primitive | How it appears |
|---|---|
| **Multi-agent supervision tree** | Five agents under a GovernedSupervisor; a crash in the coder doesn't corrupt the planner's state or the reviewer's verdict |
| **Per-agent grants** | Coder can write to `src/` — not `.env`, `Dockerfile`, CI configs, or `infrastructure/`. Reviewer cannot write at all. Test runner cannot modify source |
| **Intent declaration** | Planner declares which files will be touched and what tools will be used. Coder writing to an undeclared file triggers DriftPolicy |
| **REQUIRE_APPROVAL gate** | Destructive operations (`git push --force`, `rm -rf`, file deletes, production deploys) require human approval before execution |
| **Trust-gated autonomy** | Trust > 0.8: coder writes and commits autonomously. 0.5–0.8: all writes require review approval. < 0.5: plan-only mode, human implements |
| **Credential scoping** | Each agent gets a scoped token — test runner's token can execute `pytest` but has no git write access. Coder's token can write files but cannot push to protected branches without approval |
| **Context budget** | Per-session token cap prevents the infinite retry loop — agent summarizes and checkpoints rather than burning $80 retrying the same approach |
| **Drift detection** | Agent asked to fix a test but starts refactoring adjacent modules — drift from declared intent triggers WARN or BLOCK |
| **Audit trail** | Every file write, shell command, LLM call, and git operation logged with agent identity, reasoning, and policy state at time of action |

---

## Architecture

```
Bug report / task description
    └── GovernedSupervisor (Presidium)
            ├── Planner Agent
            │       ├── grants: [llm:claude-opus, file:read:*]
            │       ├── IntentDeclaration → PolicyEngine
            │       │       "I will read the codebase, identify the bug, and produce
            │       │        a plan specifying which files to modify and how."
            │       └── Output: structured plan (files, changes, test strategy)
            │
            ├── Coder Agent
            │       ├── grants: [llm:claude-sonnet, file:read:*, file:write:src/*,
            │       │            file:write:tests/*]
            │       ├── IntentDeclaration (inherited from plan)
            │       │       "I will modify only the files listed in the plan."
            │       ├── DriftPolicy: file:write outside plan → BLOCK
            │       ├── TrustScore check → determines approval requirement
            │       └── Output: file edits applied
            │
            ├── Test Runner Agent
            │       ├── grants: [shell:pytest, shell:npm_test, shell:make_test,
            │       │            file:read:*]
            │       ├── policy: NO file writes, NO git operations
            │       └── Output: test results + coverage diff
            │
            ├── Reviewer Agent
            │       ├── grants: [llm:claude-opus, file:read:*]
            │       ├── policy: NO file writes, NO shell execution
            │       ├── Evaluates diff against: original task, plan, test results,
            │       │   and behavioral contract
            │       └── Output: approve / request changes / reject
            │
            └── Git Agent
                    ├── grants: [shell:git_add, shell:git_commit,
                    │            shell:git_push:non-protected]
                    ├── policy: git push to protected branches → REQUIRE_APPROVAL
                    ├── policy: git push --force → DENY (always)
                    ├── CredentialVault: SSH key scoped to this repo only
                    └── Output: commit + branch push
```

---

## The Autonomy Arc

This example is designed to run across a sequence of tasks. The arc mirrors real team dynamics — a new contributor starts with close review and earns autonomy through demonstrated accuracy.

1. **First tasks:** All file writes require reviewer approval (trust starts at baseline 0.6)
2. **After 5 clean fixes (tests pass, reviewer approves with no changes):** Trust crosses 0.8 → coder writes and commits autonomously for low-risk changes (non-breaking, single-file, tests pass)
3. **One bad fix (tests fail, reviewer rejects, or drift detected):** Trust decays → all writes route back through approval
4. **Recovery:** 3 consecutive clean fixes → trust rebuilds

The governance dashboard shows this arc in real time: which tasks built trust, which eroded it, and what the current autonomy level is.

---

## Failure Modes Demonstrated

### Without governance

| Failure | What happens |
|---|---|
| Scope drift | Agent touches 14 files instead of 2. Nobody knows until the PR is reviewed manually — if it's reviewed at all |
| Test deletion | Agent deletes failing test to get green CI. Build looks healthy. Bug ships to production |
| Credential leak | Agent reads `.env`, includes API key in a commit message or sends it to an LLM |
| Force push | Agent resolves merge conflict with `git push --force origin main`. History rewritten |
| Infinite loop | Agent fails to fix a type error, retries 12 times with slight variations, burns $80 in tokens |
| Unaudited deploy | Agent runs `terraform apply` in CI. Nobody knows which agent, what reasoning, or what changed |

### With Presidium

| Failure | What Presidium does |
|---|---|
| Scope drift | DriftPolicy blocks writes to undeclared files. Audit entry records the attempt and the intent declaration it violated |
| Test deletion | Reviewer agent evaluates diff against original task. Deleting a test that was supposed to be fixed is flagged. Trust decays |
| Credential leak | GovernedToolProvider redacts secrets from tool outputs. Agent's file:read grant excludes `.env` and credential files |
| Force push | Policy: `git push --force` → DENY (always). Not a prompt instruction — the tool call is physically blocked |
| Infinite loop | Blast radius controller enforces per-session token budget and LLM call cap. Circuit breaker fires, agent suspended, developer alerted |
| Unaudited deploy | Every action tied to agent identity in registry. Production deploys gated behind HITL checkpoint. Full decision chain in audit ledger |

---

## Git-Aware Policy Primitives

This example introduces policy vocabulary specific to coding workflows. These are extensions to the standard Presidium policy engine, not replacements:

```yaml
policies:
  - name: coding-safe-defaults
    agents:
      - "coder-*"
    rules:
      # File-system boundaries
      - action: "file:write"
        decision: allow
        constraints:
          paths:
            allow: ["src/**", "tests/**"]
            deny: [".env*", "*.pem", "*.key", "Dockerfile",
                   ".github/**", "infrastructure/**"]

      # Git safety
      - action: "git:push:force"
        decision: deny
        reason: "Force push is never permitted"

      - action: "git:push"
        decision: allow
        constraints:
          branches:
            deny: ["main", "master", "release/*"]

      - action: "git:push"
        decision: require_approval
        constraints:
          branches:
            allow: ["main", "master", "release/*"]
          approval:
            min_approvals: 1
            timeout_minutes: 15

      # Shell execution boundaries
      - action: "shell:execute"
        decision: allow
        constraints:
          commands:
            allow: ["pytest", "npm test", "make test", "ruff", "mypy"]
            deny: ["rm -rf", "curl|wget * | sh", "sudo *",
                   "terraform *", "kubectl *"]

  - name: coding-budget-cap
    agents:
      - "*"
    rules:
      - action: "llm_call"
        decision: allow
        constraints:
          rate_limit:
            requests: 200
            window: "1h"
          budget:
            max_tokens: 500000
            window: "1h"
            max_cost_usd: 20.00
```

---

## Credential Scoping Detail

Each agent receives credentials scoped to its role. This is not policy — it's the token boundary. Even if policy evaluation were bypassed, the token physically cannot authorize the action.

| Agent | Token scope | Cannot do (token-level) |
|---|---|---|
| Planner | `repo:read` | Write files, push, execute shells |
| Coder | `repo:read`, `file:write:{declared_paths}` | Push to protected branches, execute arbitrary shells |
| Test Runner | `repo:read`, `shell:execute:{test_commands}` | Write files, git operations |
| Reviewer | `repo:read`, `llm:invoke` | Write files, execute shells, git operations |
| Git Agent | `repo:read`, `git:commit`, `git:push:{non-protected}` | Write source files, execute non-git shells |

---

## What This Requires from Civitas

| Dependency | Status | Notes |
|---|---|---|
| `Runtime.attach()` + `Supervisor.add_child()` | Not yet built (M4.1b) | Required if the planner dynamically spawns specialized coders. Pre-declared topology works without it |
| `GovernedToolProvider` | Designed, not shipped | Must support filesystem and shell tools, not just MCP |
| `GovernedSupervisor` | Designed, not shipped | Wraps Civitas Supervisor with policy enforcement |
| Intent declaration + DriftPolicy | Designed, not shipped | Policy engine extension |
| Trust-gated autonomy thresholds | Designed, not shipped | Registry trust score integration |

Pre-declared topology (all five agents defined in YAML) works today with Civitas alone. Governance features require Presidium implementation.

---

## Setup

> Implementation coming. Environment variables and run instructions will be here.
