---
name: sol-control
description: Use when the user explicitly invokes $sol-control for Sol-controlled, cost-aware execution of a complex, multi-part, parallelizable, or high-consequence task.
---

# Sol Control

Sol is the single controller. One or more selected workers execute bounded tasks,
verify their results, and return evidence for Sol to review.

Ordinary simple work stays direct unless the user explicitly invokes
`$sol-control`. An explicit invocation always starts with Sol. Planning-only work may use zero workers (and therefore zero Luna workers).

## Language

默认使用中文编写 Sol 计划、worker 任务与结果、状态更新和最终审核。用户明确要求
其他语言时，使用用户指定的语言。代码、命令、路径、标识符和原始证据可按需保留
原文。

## Roles

- **Sol:** understand the real goal, define `done_when`, split work, assign file
  ownership, schedule stages, and review the actual result. Sol is read-only and
  does not perform bulk implementation.
- **Luna Max:** execute exactly one assigned task, modify only its write scope,
  run the required verification, and return evidence. Luna does not redesign
  the plan, broaden scope, create subagents, or approve the overall task.
- **Terra High:** execute exactly one assigned complex task under the same
  packet, scope, evidence, and authorization rules as Luna. Terra does not
  plan, schedule, create subagents, or approve the overall task.

## Tiered execution routing

Sol is the **only controller** and final reviewer. This is a routing rule, not
a permanent agent team.

- Route to **Luna Max** only when the work is clear, low-ambiguity,
  falsifiable, small context, mechanical, or high-throughput.
- Route to **Terra High** for cross-module work, long-context investigation,
  ambiguous debugging, shared interface judgment, or high-risk implementation.
  When these traits are visible at planning time, route directly to Terra; do
  not trial Luna first merely to reduce cost.
- Start every custom agent with a fresh context: set `fork_turns="none"` and use
  the first turn only as an identity handshake. The authoritative Host/tool
  contract plus the parent launch record must prove the selected `agent_type`,
  fork mode, model, and reasoning effort. The child is not asked to self-report
  runtime identity that its surface cannot observe; it reports the effective
  permission boundary, its operational constraint, and that it performed no
  task, write, or subagent launch. No task execution or file write is allowed
  during this handshake.
- After the combined proof matches the expected custom-agent configuration,
  send the complete minimal plan or task packet to that same agent. Sol must be
  operationally read-only: require either an enforced read-only sandbox or a
  Host-owned before/after changed-path check proving zero Sol writes. Never
  combine a custom `agent_type` with a full-history fork; a full-history
  custom-agent fork is invalid and fails closed. Configuration text, an agent
  label, or a child's unsupported identity claim is not authoritative proof. If
  the Host/tool contract does not expose the exact role-to-model mapping, or if
  the exact model identity, reasoning effort, selected agent, fork mode,
  permission boundary, or no-write proof is mismatched or unprovable, do not
  send the task: **Fail Closed** and return `BLOCKED`.
- One file has one owner for the whole run. Only when Luna's first failure
  happens before Luna writes any owned file may Sol escalate the same task and
  unchanged scope to Terra once. If Luna has written any owned file before
  failing, Luna retains all ownership; Terra never replaces that owner. Terra's
  write state is never the escalation gate.

## Workflow

1. Sol writes the smallest useful plan.
2. Independent tasks with disjoint write scopes may run in the same stage.
3. Dependent or overlapping tasks run in later stages.
4. Luna Max or Terra High executes and self-checks each assigned task.
5. Sol reviews real files, the diff, test or build output, and requirement
   coverage before deciding `PASS`, `FIX`, or `BLOCKED`.

## Execution continuity and planning convergence

For authorized execution, a plan is not a stop point. Stop or pause only for a
new permission request, an irreversible choice requiring confirmation, or a
real blocker, or an explicit user cancellation, replacement, or redirection of
the current request; these are the only stop gates. Otherwise continue through
the approved stages.

An explicit user cancellation, replacement, or redirection stops the old plan
and requires re-planning from the new request. Substantive user steering is not
an ordinary status inquiry; do not continue old-plan execution while Sol
re-plans.

An ordinary status question or status inquiry does not pause authorized work;
it requires no new permission and is not a blocker. Report the current state
and continue the existing plan and evidence loop.

Sol's planning has a host-stated planning timebox. Within the planning timebox,
Sol must converge to a plan, a determination, or a concrete evidence gap. The
plan, determination, or evidence gap must be produced before the planning
timebox ends. Extended analysis without convergence is not progress.

If a later or downstream stage is blocked, deliver the earlier or prior stage
when it is evidence-complete, including its artifact and evidence. Partial
delivery is allowed only when the completed stage is evidence-complete; only
the unresolved downstream work remains blocked.

## Sol plan

```yaml
goal: "The user's final outcome"
done_when:
  - "An observable completion criterion"
tasks:
  - id: task-a
    task: "One clear task"
    write_scope: ["owned/path"]
    do_not_touch: ["all other paths"]
    expected_result: "What must be observable"
    verification: "Exact command or procedure"
    context: "Optional relevant input"
stages:
  - [task-a]
```

Sol uses the minimum number of selected workers needed. A stage may launch only as
many ready tasks as live capacity allows; excess tasks wait in the next batch.
No fixed worker maximum is promised by this Skill.

## Shared execution task

Every delegated task uses this complete packet. `Context` is optional; every
other field is required.

```text
Task ID: <stable task id>
Task: <one bounded task>
Context: <optional files, evidence, or prior-stage result>
Write scope: <exact writable paths>
Do not touch: <excluded paths and side effects>
Expected result: <observable acceptance condition>
Verification: <exact command or procedure and passing condition>
```

An incomplete, contradictory, or unauthorized packet is `BLOCKED`; the worker must
not guess the missing scope.

## Shared execution result

```text
Task ID: <task id>
Status: PASS | FIX | BLOCKED
Summary: <what happened>
Changed: <exact files, or None>
Verification: <commands and exact results>
Evidence: <diff, test, build, log, or artifact evidence bound to the final candidate>
Repair attempt: 0 | 1 | 2 | 3
Progress: <evidence-based comparison with the previous result, or None>
Failure class: runtime | model_identity | permission | dependency | scope | verification | conflict | none
Blocker: <None or the concrete blocker>
```

`FIX` is an internal continuation state, not a final delivery. Ordinary
development failures—failed tests, builds, compilation or type checks,
incomplete implementation, missing regression coverage, wrong fields, or
incorrect CLI behavior—are `FIX`, not `BLOCKED`. `BLOCKED` means that work
cannot continue: permission or authorization is missing, required credentials
or dependencies are unavailable, model identity cannot be proved, requirements
conflict, ownership cannot be resolved legally, or continuing would expand
scope without authorization. Do not use `BLOCKED` merely because the work is
unfinished.

Evidence must bind to the final candidate identity, represented by a commit+diff
identity or an exact changed-file snapshot. If the candidate changes after
verification, prior evidence is stale and affected verification must be rerun
before `PASS`. A top-level `Candidate` result field is not added.

transport/spawn `completed` only proves delivery lifecycle completion; it cannot substitute for a structured Luna `PASS` or internal `FIX`, or a structured Terra `PASS` or internal `FIX`, Verification/Evidence/changed-path proof, or Sol review.

If transport/spawn reports `completed` without a structured result, allow
exactly one result-only follow-up to the same worker. This result-only follow-up
authorizes no new write and no re-execution. If it still cannot retrieve a
structured result bound to the final candidate, return `BLOCKED`; do not launch
another retrieval.

The worker may return `PASS` or internal `FIX` for its assigned task only. Sol
decides whether the overall work is complete.

## Scheduling and ownership

- One file has one owner for the entire run.
- Multiple workers may read the same file, but they must not write it
  concurrently.
- A shared integration file has one worker owner.
- If write scopes overlap or the overlap is uncertain, merge the tasks or
  schedule them sequentially.
- Preserve unrelated uncommitted user changes and verify the final real diff.

## Review and correction

Sol returns `PASS | FIX | BLOCKED`. Evidence-free `PASS`, out-of-scope writes,
failed verification, conflicts, or missed criteria cannot pass review. A normal
defect enters a bounded repair loop of at most three focused repairs. Every
repair keeps the original owner and original write scope, and must be based on
new verification evidence rather than a repeated prompt. Continue only when
there is material progress; stop early when the core failure is unchanged. After
three unsuccessful repairs, return `BLOCKED`.

Every Correction Packet keeps the original owner and original scope, and contains
`Failure class: runtime | model_identity | permission | dependency | scope | verification | conflict | none`
plus `Attempt: 1 | 2 | 3`, a same-scope `Delta`, `Expected progress`, and the
latest `Progress` comparison. The `none` class means no failure occurred; any
failure uses another allowed class. The same task packet with no new evidence is `BLOCKED` and is not relaunched.

If the failure shows that the decomposition—not the implementation—is wrong,
Sol may re-plan automatically while the user's goal, total authorization, and
do-not-touch boundaries remain unchanged, no new dangerous or irreversible
operation, credential, or user decision is needed, and no ownership conflict is
created. Files already written retain their owner; unwritten files may be
reassigned by the re-plan. A new authorization, credential, dangerous operation,
explicitly excluded path, or legally unresolved ownership is a true `BLOCKED`
condition.

User urgency, requests to hurry, or saying "do not stop" cannot lower, relax,
or reduce the evidence or verification threshold. The evidence threshold
remains unchanged and every safety gate still applies.

## Resume packet (long tasks only)

Resume packets are only for tasks expected to cross context compression, suffer a
session interruption, or run for a long time. The minimal packet contains only
`goal`, `completed`, `in_flight`, `artifact_location`, and `next_action`. Short and Direct tasks never generate a resume packet.

Deletion, deployment, production changes, accounts, payment, credentials, or
other external side effects require explicit user authorization before work
begins.

See [orchestration.md](references/orchestration.md) for the operating contract
and [runtime-notes.md](references/runtime-notes.md) for internal dispatch rules.
