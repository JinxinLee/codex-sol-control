# Sol Control orchestration contract

This reference defines the operating contract. Sol owns every scheduling and
completion decision; Luna Max or Terra High owns bounded execution.

## 1. Start and route

- Explicit `$sol-control` invocation starts Sol.
- The Host starts `sol-controller` with `fork_turns="none"` for an identity-only
  handshake. The authoritative Host/tool contract and launch record prove the
  requested `agent_type`, fork mode, model, and reasoning effort. Sol reports
  its effective permission boundary, operational read-only constraint, and zero
  task/write/subagent activity without planning or writing. The child is not
  asked to self-report runtime identity that the surface cannot expose. Only
  after the combined proof matches does the Host send the plan request to that
  same Sol. Every worker uses the same two-turn handshake before it receives a
  task packet. A full-history custom-agent fork is invalid and fails closed.
- Ordinary simple work without explicit invocation remains direct.
- Planning-only or review-only work may stop after Sol and use zero workers (and therefore zero Luna workers).
- Execution work uses the minimum useful number of workers selected by Sol.

Sol is the only controller and final reviewer; this is not a permanent agent
team. Route Luna Max only to clear, low-ambiguity, falsifiable, small context,
mechanical, or high-throughput work. Route Terra High to cross-module work,
long-context investigation, ambiguous debugging, shared interface judgment, or
high-risk implementation. When those traits are visible at planning time, route
directly to Terra rather than trialing Luna first to reduce cost. Terra never
plans or approves the overall task.

An invoking Skill may attach an explicit mode block to Sol's post-handshake plan
request. With `Execution mode: luna_only` and `Allowed workers: luna-max-worker`,
Terra is unavailable: Sol must not dispatch or escalate any task to Terra and
must decompose or re-plan complex work into bounded Luna tasks. Without that
mode, the normal `$sol-control` Luna/Terra routing and zero-write escalation
remain available.

Before task execution or any write, combine the authoritative Host/tool role
mapping, parent launch record, and child's permission/no-side-effect handshake
to prove exact model identity, reasoning effort, selected custom agent, fork
mode, and effective inherited permission boundary. Configuration text, an agent
label, or a child's unsupported identity claim is not authoritative proof. Sol
must have either an enforced read-only sandbox or a Host-owned before/after
changed-path check proving zero Sol writes after every Sol turn. If any required
proof is unavailable or mismatched, do not send the task: **Fail Closed** and
return `BLOCKED` rather than substituting a nearby model or silently weakening
the read-only boundary.

For authorized execution, a plan is not a stop point. Stop or pause only for a
new permission request, an irreversible choice requiring confirmation, or a
real blocker, or an explicit user cancellation, replacement, or redirection of
the current request; these are the only stop gates. Ordinary status questions or
status inquiries do not pause authorized work, require no new permission, and are
not blockers.

An explicit user cancellation, replacement, or redirection stops the old plan
and requires re-planning from the new request. Substantive user steering is not
an ordinary status inquiry; do not continue old-plan execution while Sol
re-plans.

Sol's planning has a host-stated planning timebox. Within that planning timebox,
Sol must converge to a plan, a determination, or a concrete evidence gap. The
plan, determination, or evidence gap must be produced before the planning
timebox ends; extended analysis without convergence is not progress.

When a later or downstream stage is blocked, deliver an earlier or prior stage
that is evidence-complete with its artifact and evidence. Partial delivery is
allowed only when the completed stage is evidence-complete; only unresolved
downstream work remains blocked.

## 2. Sol plan

Sol produces only this top-level shape:

```yaml
goal: "Concrete outcome"
done_when:
  - "Observable criterion with evidence"
tasks:
  - id: task-a
    task: "One bounded action"
    write_scope: ["exact/path"]
    do_not_touch: ["excluded/path"]
    expected_result: "Observable result"
    verification: "Exact command or procedure"
    context: "Optional input"
stages:
  - [task-a, task-b]
  - [task-c]
```

`context` is optional. All other task fields are required. Each `done_when`
criterion must map to inspectable evidence or an exact verification.

## 3. Stages and live capacity

Task IDs in one stage are independent and may run concurrently only when their
write scopes are disjoint. Later stages wait for earlier stages to finish and
pass review. At every launch, the dispatcher checks live capacity and starts
only the ready tasks that fit. Remaining tasks stay queued for another batch;
the plan never assumes a fixed worker count.

If dependency order or write overlap is uncertain, schedule sequentially.

## 4. One file, one owner

Every writable file has one owner for the entire run. Read-only analysis may be
parallel, but two workers never modify the same file. A shared integration
file also has one owner. Alternative proposals may be collected read-only;
after Sol selects a proposal, one worker performs the write.

Before launch, reject a stage that contains overlapping write scopes. Before
integration, compare the real changed paths with every assigned scope and
preserve unrelated dirty-worktree changes.

## 5. Shared execution task

```text
Task ID: <stable task id>
Task: <one bounded task>
Context: <optional files, evidence, or prior-stage result>
Write scope: <exact writable paths>
Do not touch: <excluded paths and side effects>
Expected result: <observable acceptance condition>
Verification: <exact command or procedure and passing condition>
```

If a required field is missing, incomplete, or contradictory, or the packet does
not make the execution boundary determinable, Luna or Terra returns `FIX` with
the concrete packet defect without writing, guessing, or broadening scope. The
worker judges only packet sufficiency and does not decide whether Sol can repair
or re-plan within total authorization; a packet defect is not worker `BLOCKED`.
Independent execution blockers after packet sufficiency, such as an absent
dependency or unproved authorization, remain `BLOCKED`.

## 6. Shared execution result

```text
Task ID: <task id>
Status: PASS | FIX | BLOCKED
Summary: <what happened>
Changed: <exact files, or None>
Verification: <commands, exit status, and concise output>
Evidence: <diff, test, build, log, or artifact location bound to the final candidate>
Repair attempt: 0 | 1 | 2 | 3
Progress: <evidence-based comparison with the previous result, or None>
Failure class: runtime | model_identity | permission | dependency | scope | verification | conflict | none
Blocker: <None or the concrete blocker>
```

`FIX` is an internal continuation state. Ordinary development failures such as
failed tests, builds, compilation or type checks, incomplete implementation,
missing regression coverage, wrong fields, or incorrect CLI behavior enter
`FIX`. `BLOCKED` is reserved for a true stop: missing permission,
authorization, credentials, or required dependencies; unprovable model
identity; contradictory requirements; legally unresolvable ownership; or an
unauthorized scope expansion. `BLOCKED` means unable to continue, not merely
unfinished.

`PASS` requires every assigned acceptance condition and verification to be
evidenced. Luna or Terra approves only its bounded task; neither approves the
overall project.

Evidence must bind to the final candidate identity using either a commit+diff
identity or an exact changed-file snapshot. If the candidate changes after
verification, the old evidence is stale: enter `FIX` and rerun affected
verification. Stale evidence cannot pass. If required reverification is
unavailable because of a real dependency, permission, or environment blocker,
return `BLOCKED`.
Do not add a top-level `Candidate` field to the worker result.

Transport/spawn `completed` only proves delivery lifecycle completion. It cannot
substitute for a structured Luna `PASS` or internal `FIX`, or a structured Terra `PASS` or internal `FIX`, Verification/Evidence/changed-path proof,
or Sol review.

If transport/spawn reports `completed` without a structured result, allow exactly
one result-only follow-up to the same worker. The result-only follow-up may not
authorize a new write or re-execution. If it still cannot retrieve a structured
result bound to the final candidate, return `BLOCKED`; do not launch another
retrieval.

## 7. Sol review

Sol inspects the original request, `done_when`, real files, complete diff,
verification output, build or artifact results, and every worker result. It also
checks that tasks integrate cleanly and unrelated user edits remain intact.

Sol returns exactly one verdict:

- `PASS`: all completion criteria and required evidence are satisfied.
- `FIX`: one concrete defect can be corrected inside the original owner's
  unchanged scope.
- `BLOCKED`: permissions, dependencies, runtime selection, scope, conflicts,
  or verification prevent a defensible completion claim.

Sol must reject an evidence-free worker `PASS`, an out-of-scope write, an omitted
criterion, or a failed command.

In normal tiered mode, only when Luna's first failure happens before Luna writes
any owned file may Sol perform one bounded escalation of the same task and
unchanged scope to Terra instead of unbounded Luna retry attempts. If Luna has written
any owned file before failing, Luna retains all ownership; Terra never replaces
that owner. Terra's write state is never the escalation gate. In `luna_only`
mode, this escalation is disabled completely. The packet, authorization
boundary, evidence freshness, correction rules, and scope remain unchanged.

## 8. Bounded repair and authorized re-plan

An ordinary defect enters a bounded repair loop of at most three focused repairs:

```text
Task ID: <original-id>-fix
Attempt: <1|2|3>
Failure class: <runtime|model_identity|permission|dependency|scope|verification|conflict>
Issue: <observed defect and evidence>
Delta: <new same-scope instruction or new evidence; never an unchanged prompt>
Expected progress: <measurable change expected in the next verification>
Scope: <the original owner's unchanged write scope>
Verification: <exact regression command or procedure>
Progress: <evidence-based comparison after the repair, or None before it runs>
```

The same owner performs every repair. Continue only when verification shows
material progress; stop early when the core failure is unchanged. After three
unsuccessful repairs, return `BLOCKED`. An identical packet with no new evidence
is not relaunched and is `BLOCKED`. The `none` failure class means no failure;
any failure uses another class.

If a worker returns `FIX` for a packet defect, Sol owns the final repairability
decision. Sol compares the original goal, `done_when`, total authorization,
`do_not_touch`, permission, ownership, and user instructions. When those
constraints safely determine a correction, Sol issues a fresh corrected packet
and redispatches it to the same owner; otherwise Sol returns `BLOCKED`.

If the evidence shows that the decomposition—not the implementation—is wrong,
Sol may automatically re-plan within the same unchanged constraints when no
dangerous or irreversible operation, credential, or new user decision is needed
and no explicit `do_not_touch` boundary is violated. Files already written
retain their owner; unwritten files may be reassigned. New authorization,
credentials, dangerous operations, excluded paths, or an unresolvable ownership
conflict are true `BLOCKED` conditions.

User urgency, requests to hurry, or saying "do not stop" cannot lower, relax, or
reduce the evidence or verification threshold. The evidence threshold remains
unchanged and every safety gate still applies.

The focused correction is a Correction Packet with the original owner and
unchanged scope. It must include `Failure class` from exactly `runtime`,
`model_identity`, `permission`, `dependency`, `scope`, `verification`, `conflict`,
or `none`, plus a `Delta` that changes the same-scope task packet or adds new
evidence. The `none` class is valid only when no failure occurred; any failure
uses another allowed class. An identical task packet with no new evidence is
`BLOCKED` and must not be relaunched.

## 9. Resume packet (long tasks only)

Resume is only for tasks expected to cross context compression, be interrupted,
or run for a long time. Its minimal packet contains only `goal`, `completed`,
`in_flight`, `artifact_location`, and `next_action`. Short and Direct tasks do not
generate a resume packet.

## 10. Safety boundary

The dispatcher cannot widen user authorization or the parent permission
boundary. Deletion, deployment, production changes, accounts, payment,
credentials, and external side effects require explicit authorization. No
result summary substitutes for inspection of real artifacts and evidence.
