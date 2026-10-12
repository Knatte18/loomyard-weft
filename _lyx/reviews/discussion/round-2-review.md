---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: skip_alignment is classified open to the local layer and run_settings, but a local or run_settings false overrides a hub-level true and re-opens the agent's auto-landing approval, which meets PATTERN-config-layers' Guard category
    fix: In D7 (decision-record.md:123-124), move `skip_alignment` to the closed set beside `alignment` and `alignment_pr_approve`, with the reason that a local `false` under a hub `true` hands both gates back to the agent and re-opens the PR-Gate landing; keep only `alignment_timeout_min` open, bounded by the coherence check. Update the Constraints line on `PATTERN-config-layers` only if its wording no longer matches, and change the acceptance criterion at decision-record.md:194 so a local-layer or `run_settings` `skip_alignment` is refused and only `alignment_timeout_min` is accepted.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: D8's route for a decision on a run-held task fails before Discussion-Write starts and for every darn run, and its premise that the verb's refusal points to a message to the Discussion-Write session holds only while that row is running
    fix: "In D8 (decision-record.md:134-137), replace the clause after the semicolon with a route that exists in every window: when `lyx loom decision add` refuses because the run has no decision record (batten's pre-loom rows such as `running · Worktree-Create`, `Preflight`, `Loom-Preflight`, a halt before Discussion-Write, or any darn run), the orch records the decision as a board note of its own through `lyx board upsert` with kind note naming the held task's slug, which the run lock allows and which D2 already has the agent read through `lyx board list`; keep the message to the Discussion-Write session only for the refusal that names it, while that row is running. Name the note route in D8's Rationale and in D5's bound sentence."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:96
    summary: D5's bound rests on D8's recording rule, but an operator decision recorded through decision add while PR-Align's agent runs, after it read the record, is never seen and the agent's approve still lands
    fix: In D5, add to the verdict check that Go records the decision record's content hash (or the records commit) at spawn and applies an `approve` only when it is unchanged at apply time, applying it as an escalation otherwise; state that window in D5's Bound beside the head and pull request check, and add an acceptance criterion that a decision added during `PR-Align` turns the agent's approve into an escalation. Apply the same check to D4's parent-review approve if the parent-review route in F2 can change the record mid-review.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:80
    summary: D5 and the PR-Gate acceptance criteria call the approval or rejection record a decision record, the name D2, D6 and D8 use for decision-record.md, which D8 makes the operator-decision channel
    fix: In D5 (decision-record.md:80 and :86) and the acceptance criteria at decision-record.md:184 and :187, replace "decision record" meaning the approve or reject file with "approval or rejection record", leaving "decision record" only for `decision-record.md`.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:113
    summary: D6's Bound lists what the verb reads as the decision record, plan and changed-path list, while its Decision also prints each peer's title, status and parent branch and the parent's commits since the run's base
    fix: In D6's Bound (decision-record.md:113), state the read set as the Decision gives it, adding the board fields and the parent's commit list (short SHA and subject), or narrow the Bound to the record content `show` prints and say the default form's other fields are board and git metadata only.
---

### [BLOCKING] `skip_alignment` open to the local layer re-opens the auto-landing gate

**Location:** `_lyx/discussion/decision-record.md:123`

**Issue:** D7 opens `skip_alignment` to the local layer and `run_settings` on the reasoning that "the off switch only routes a review to the orch".
That holds for the value `true` only.
A local layer holds a key at any value, so when the hub's `loom.yaml` sets `skip_alignment: true` to keep both points with the orch, a pair's local file or a task's `run_settings` setting `false` puts the agent back at both points, and with `alignment_pr_approve` at its default `true` its approve lands on the parent and runs `deploy` with no orch turn (D5's Bound).
`pattern/PATTERN-config-layers.md` tells the classifier to ask "what a key's value can do to a guard, not what kind of key it is", and the Guard category covers a key that "switches off or hollows a guard, including a review or an approval gate".
D7 applies that test to `alignment_pr_approve` ("a re-opened auto-landing hollows the approval gate") but not to the switch that re-opens the same landing from the other direction.
`run_settings` reaches a run-held entry, since `boardengine.scopeChanged` never compares `RunSettings` (`internal/boardengine/runlock.go`).
This is a violation of a listed constraint, hence BLOCKING.

### [MEDIUM] D8's route for a held task's decision has windows with no route

**Location:** `_lyx/discussion/decision-record.md:136`

**Issue:** D8 sends a decision on a run-held task to `lyx loom decision add`, and "before Discussion-Write has written the record, the decision goes by message to the Discussion-Write session, as the verb's refusal says".
In `internal/loomcli/decision.go` the verb has two distinct refusals here.
While Discussion-Write is `running` it refuses with `decisionWayWriter`, which does say to message the Discussion-Write session.
When the record does not exist and that row is not running, it refuses with `decisionWayNoRecord`: check `lyx loom status`, run `lyx loom start`, and for darn "name the call in your report to the run's parent", and the orch is that parent.
So the premise is false for the window from batten's claim (`boardengine.RunStatus(running, Worktree-Create)` in `internal/battencli/wire.go`) through `Preflight` and `Loom-Preflight`, for a halt in that window, and for every darn run, which never has a record.
In those windows the board body is locked (`checkRunLocks` refuses any change but `Status`), no record exists and no Discussion-Write session exists to message, so a decision has nowhere the agent can read.
The run lock's own refusal names a route that works in every window: a board note of its own through `lyx board upsert` with kind note, which the agent already reads through `lyx board list` (D2).
D5's Bound leans on D8's rule, so the gap widens what an agent approval lets through.

### [MEDIUM] D5's bound misses a decision recorded during `PR-Align`'s wait

**Location:** `_lyx/discussion/decision-record.md:96`

**Issue:** D5 bounds what an agent approval lets through to "a conflict with an operator decision the agent cannot read, bounded by D8's decision-recording rule".
D8 has the orch append a running task's decision to its decision record through `lyx loom decision add`, which commits through `fabricengine.CommitAnchoredPaths` into the records worktree (`internal/loomcli/wiring.go`), so neither the pull request head nor the local task head that `PR-Gate` compares (`internal/landingshed/prgate.go`, `TaskHead` from `fabricengine.HeadSHA` in `internal/loomcli/landingdeps.go`) moves.
An operator decision the orch records while the agent runs, after the agent read the record, therefore passes D5's head and pull request check, and the agent's approve lands with up to `alignment_timeout_min` of such window.
An orch approval has no such window, since the orch makes the decision it approves against.
D5 names no check that closes it, so the bound as stated is wider than D8 makes it.

### [LOW] "decision record" names two different files

**Location:** `_lyx/discussion/decision-record.md:80`

**Issue:** D5 skips "when a decision record already matches the pull request and head" and writes "no decision record" on an escalation, and the acceptance criteria say "no decision record" for the PR-Gate approve with `alignment_pr_approve: false` and the PR-Gate escalate.
Those mean the approval or rejection record `PR-Gate` reads.
D2, D3, D6 and D8 use "decision record" for `decision-record.md`, which D8 makes the channel for operator decisions, so a plan writer can read D5 as checking or withholding the discussion artifact.

### [LOW] D6's Bound narrower than its Decision

**Location:** `_lyx/discussion/decision-record.md:113`

**Issue:** D6's Bound says the verb "reads only tasks whose board status is a run status and only their decision record, plan and changed-path list".
The Decision's default form also prints each peer's title, status and parent branch, and the parent's commits since the caller's merge base.
The Bound should state the read set the Decision gives, so a plan writer does not drop the commit list to fit it.

## Focus verdicts

- D8 and D5 against the run lock: the run lock refuses any change but `Status` and `RunSettings` on a held entry, as D8's Rationale says, and `decision add` writes the record rather than the board, so its route is carried out wherever the verb accepts; it fails in the windows F2 names, and D5's bound does not hold across the in-flight window F3 names.
- D3 and D6 peer set: not a defect.
  D6 defines the set once, as every task whose status `boardengine.IsRunStatus` accepts, whatever the state, the caller excluded; D3 cites "D6's peer set", and `show`'s refusal and the Bound use the same test.
  `boardStatus` in `internal/loomcli/wiring.go` writes `RunStatus(state, producer)` for every state but `done`, so awaiting and blocked peers are run statuses and included, and the acceptance criterion pins one.
- D5 and D7 limits: `merge_auto_accept` exists in `internal/loomengine/config.go` as `CategoryGuard`, and a Merge-Review settle carries open risks into the decision record through the CarryOver seam (`internal/shedadapters/bouncercarryover.go`), which the orch stencil's PR-Gate fork reads; `CategoryGuard` and `CategoryNone` are the two categories `ConfigSpec` uses, and `arbiter` and `review` are `none` as D7 says.
  The one bound that does not hold is `skip_alignment`'s, F1.
