---
name: sol-luna
description: Use when the user explicitly invokes $sol-luna for Sol-controlled execution restricted to Luna Max workers.
---

# Sol Luna

`$sol-luna` is a formal personal specialization that inherits the complete `$sol-control`
contract, not a second orchestration implementation. Read
`../sol-control/SKILL.md` and follow
its complete contract, including the referenced orchestration and runtime notes.
The rules below are the only mode-specific routing override and take precedence
over the base tier-selection rule when this Skill is explicitly invoked.

## Sol handoff

After Sol's identity-only handshake, include this exact mode block in Sol's
plan request; do not rely on implicit Skill text:

```text
Execution mode: luna_only
Allowed workers: luna-max-worker
Terra High: unavailable
Routing constraint: no Terra dispatch or Luna-to-Terra escalation
```

Complex work still requires Sol to decompose or re-plan into bounded Luna tasks.

## Luna-only routing

- Sol remains the only controller and final reviewer; all planning, ownership,
  scheduling, evidence checks, repair, and authorized re-planning stay intact.
- Terra High is unavailable in this mode. Sol must not dispatch any task to
  Terra; every delegated execution task goes to one or more Luna Max workers.
- Parallel Luna workers still require disjoint write scopes. Shared-file or
  dependent tasks run in stages, with one owner per file for the whole run.
- If work would normally use Terra, first decompose or re-plan it into bounded,
  Luna-suitable tasks without expanding authorization, weakening verification,
  or transferring written-file ownership. Do not pass the original unbounded
  task to Luna and do not return `BLOCKED` merely because it is complex.
- Return `BLOCKED` only when safe decomposition cannot continue within the
  existing authorization, scope, ownership, and evidence contract.

Keep explicit invocation and `allow_implicit_invocation: false`; do not copy/redefine base rules; inheritance from `$sol-control` is authoritative.
