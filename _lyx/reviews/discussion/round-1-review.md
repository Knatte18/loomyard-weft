---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:43
    summary: D1 routes timed-out and unstartable verifies to the fixer round, but Publish writes no failure record for either, so the round gets nothing to fix and the premise is false for two of the three named cases
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:69
    summary: D3 narrows the guard only for a halted run, but start also refuses a run still running at Publish or Finalize whose driver died mid conflict session, which is the crash leftover the resolver's own abort exists for
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:103
    summary: D6's premise that only the parent and the operator are slug-free is false in a standalone run, where every agent's strand name is slug-free and passes the check, and the new plan-stencil text names a parent that run does not have
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:97
    summary: D6 writes the literal ly:orch into the refusal way forward and the shipped plan stencil, but the shortname is per hub, so every other hub would name a session that does not exist
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:91
    summary: D5's No code changes unless the scenario fails leaves the fail branch undesigned, though the code already reads stopped as terminal, so the Decision is a test-only one and should say so
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: D1's Bound says Publish reruns both verifies in full, but verifytree skips the plan verify when the round gate already passed the plan verify command on the same tree
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:66
    summary: D2's Bound omits that the parent-source test also matches a parked merge-in an operator or orch started by hand with lyx fabric merge-in, which the abort then discards
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:146
    summary: the acceptance criterion that both Webster-Review rows render the Publish failure section asserts behavior gate-test-scope already built and no Decision changes
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md:122
    summary: the last two Constraints narrate planned work on webster-replan-state and driver-context and carry no requirement the plan writer acts on
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:137
    summary: the Open risks entries for D1, D2, D3 and D6 restate those Decisions' Bounds or Out items instead of adding a risk
---

### [BLOCKING] D1 sends timeouts and unstartable verifies to a round that has no record of them

**Location:** `_lyx/discussion/decision-record.md:43` (D1), and the Goal, the first Acceptance criterion and the `Publish: verify failed` spec row that follow from it.

**Issue:** D1 makes the Stuck reason for a verify that "fails, times out or cannot start" name `goto --to Webster-Burler`, on the premise that "the Webster-Review round reads the Publish failure record and its gate runs the failing tests".
`verifyGate.recordFailure` in `internal/landingshed/verifygate.go` returns without writing when `result.TimedOut` or `result.ExitCode < 0`, the comment saying neither "names a failing test".
So for two of the three cases there is no record: `PublishFailureNote` renders `none`, the round gate in `internal/loomshed/verifygate.go` adds no `tmux` steps, and the reviewer judges the diff against the plan with no knowledge of the failure.
That is the failure #462 reports: both rounds approve with unrelated NITs and the defect is never reached.
A shell that cannot start is also an environment fault, not a code defect a fixer round can fix.

**Fix:** Restrict D1's route to a verify that ran and exited non-zero, the one case that writes a record.
For a timeout and an unstartable shell, either keep today's reasons, or decide what the record carries for each and add that write to Scope; state which in D1.
Change the Goal bullet, the first Acceptance criterion and the spec row to match.

### [MEDIUM] D3 leaves the crashed-driver leftover refused

**Location:** `_lyx/discussion/decision-record.md:69` (D3), and the matching Acceptance criterion.

**Issue:** D3 lets resume and start through only for "a run halted at Publish or Finalize".
`lyx loom start` calls `refuseOverUnfinishedMerge` whenever it must spawn a driver or wake a parked one (`internal/loomcli/start.go`), whatever the run's state.
A driver killed during Publish's conflict session leaves the run `running` at Publish with a parked merge whose `Source` is the parent branch.
That is the state the resolver's own "crash recovery" abort exists for, and D2's abort would discard it safely.
Under D3 as written, `start` still refuses it and the hub falls back to the hand `lyx fabric merge --abort` that #468 recorded.

**Fix:** Decide the crashed case in D3.
Either widen the exception to any non-done run whose current producer is Publish or Finalize, or state that a running run stays refused and why, and name its way forward.
Add the decided case to the Acceptance criterion.

### [MEDIUM] D6's slug-free premise fails in a standalone run

**Location:** `_lyx/discussion/decision-record.md:103` (D6 Rationale), and D6's Bound at line 105.

**Issue:** The Rationale says "the parent and the operator are the two slug-free callers".
`reedengine.Geometry.NameSlug` is documented as empty "in the prime and in a standalone run", and a standalone run's agents are named `s<hash>:<role>` (`agentname.StandaloneShortname`).
So every agent of a standalone run passes D6's check and can still label a call `--by parent` or `--by operator`, which is the #453 defect.
A standalone run also has no parent, yet D6's new plan-stencil text tells its plan agent to ask the orch to record the call.
The Bound covers a cleared variable, but not this let-through.

**Fix:** Decide the standalone case.
Either refuse any non-empty strand name when the run has no recorded parent, or state in the Bound that the check guards hub task sessions only and lets a standalone run's agents through.
Make the plan-stencil text correct for a run with no parent too, for example by recording the call as an open question in the plan.

### [MEDIUM] D6 hard-codes this hub's shortname

**Location:** `_lyx/discussion/decision-record.md:97` and `:100` (D6).

**Issue:** The way forward names "the parent, `ly:orch`", and the plan-stencil replacement says "ask `ly:orch` to record it".
`ly` is this hub's shortname.
The shortname is read per hub (`fabricengine.ReadShortname`), the parent's name is resolved per task (`hubgeom.ResolveParent`), and stencils are seeded into every hub.
In any other hub, the stencil and the refusal would name a session that does not exist.

**Fix:** In the stencil, say "your parent", which the rendered parent directive names.
In the refusal, either say "the parent session" or interpolate the resolved parent name.

### [LOW] D5's conditional leaves a branch undesigned

**Location:** `_lyx/discussion/decision-record.md:91` (D5).

**Issue:** "No code changes unless the scenario fails" defers the fix for the fail case to the plan writer, and the writer's stencil requires designing the full scope.
The branch cannot be reached anyway: `isTerminalOutcome` in `internal/shuttleengine/attach.go` already lists `runOutcomeStopped`, and `leftoverThenAgeVerdict` returns respawn-eligible for any terminal outcome before the age check.
`TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge` covers the untracked and binding-cleared paths for done, asking, died and timeout, but not stopped.
The pass outcome in the Acceptance criteria is unambiguous; the fail outcome is unstated.

**Fix:** Make D5 a test-only Decision: add `stopped` to the existing attach and burler-probe tables, and change no code.
Drop the conditional.

### [LOW] D1's Bound overstates Publish's re-verify

**Location:** `_lyx/discussion/decision-record.md:54` (D1 Bound, second sentence).

**Issue:** The Bound says "Publish reruns both verifies in full on its next pass".
`verifyGate.check` goes through `verifytree.Verify`, which returns `StatusSkipped` when the record already names HEAD's tree under the same command.
Where impactset falls back, the Webster-Burler round gate runs the plan's own verify command and keeps that record entry.
If the parent has not moved, Publish then skips the plan verify.
The safety claim survives, because the same tree passed the same command, but the stated mechanism is false.

**Fix:** State it as: Publish verifies again, skipping only a tree its record already names under the same command, and removes the failure record only after both verifies pass or skip.

### [LOW] D2's Bound misses the hand-started parent merge-in

**Location:** `_lyx/discussion/decision-record.md:66` (D2 Bound).

**Issue:** Besides the landing rows, the only `MergeIn` caller is `lyx fabric merge-in` (`internal/fabriccli/merge_verbs.go`).
A parked `lyx fabric merge-in <parent>` that an operator or orch starts by hand in the task pair carries the same `Source`.
D2's abort discards it, including any partial resolution.
The Bound names only "a conflict session's resolution".
D3's Bound names only the orch at a Publish halt, so the operator case is unstated, and that is the attack-surface question the rubric asks.

**Fix:** Add to D2's Bound that any parked merge of the parent branch into the task pair is discarded at Publish or Finalize entry, whoever started it.

### [LOW] An acceptance criterion pins already-built behavior

**Location:** `_lyx/discussion/decision-record.md:146`.

**Issue:** "Both Webster-Review rows, the Bouncer judge and the Burler reviewer, render the Publish failure section from the record" traces to no Decision.
It is already built:
`shedadapters` Bouncer fills the rubric's `publish_failure` marker (`publishFailureNote`), `shedrecipe`'s `burlerRoundProfile` fills it on the stencil route, and `contracts/stencils/rubric_test.go` pins the marker.
Scope's Out puts the failure record's shape under `gate-test-scope`.

**Fix:** Drop the criterion.
The Notes line asking the plan writer to confirm the Burler marker already covers the dependency.

### [NIT] Two Constraints carry no requirement

**Location:** `_lyx/discussion/decision-record.md:122-123`.

**Issue:** The `webster-replan-state` and `driver-context` bullets describe other entries' planned work.
They conclude "the two do not overlap" and "stays a hub command until then", which neither constrains the plan nor adds to D1 and Scope's Out.
Interview turn 1 of the support log already records both entries.

**Fix:** Delete both bullets.

### [NIT] Open risks restate Bounds

**Location:** `_lyx/discussion/decision-record.md:137-141`.

**Issue:** The D2, D3 and D6 risks repeat their Decisions' Bounds almost word for word, and the D1 risk repeats Scope's Out and D1's Bound.
Only the D4 entry adds a risk stated nowhere else.

**Fix:** Keep the D4 entry and delete the others, or reduce each to a pointer to its Decision's Bound.

## Focus verdicts without a finding

- D2 and D3 premise: holds.
  `MergeIn` records the `source` argument as `mergeState.Source`, and Publish and Finalize both pass `Deps.ParentBranch` through `Resolver.Resolve` unchanged.
  The landing rows and `lyx fabric merge-in` are the only `MergeIn` callers, so the test neither misses the row's own leftover nor matches a merge another lyx component started; F7 covers the hand-started case.
  The status commit that runs before the clean check commits only the records side (`CommitRecordsPaths`), which is not a merge-in participant, so a parked warp merge does not stop it first.
- D1 route: reachable.
  `shedengine.Goto` leaves the run paused at the target row, `resume` wakes the parked driver, and Webster-Burler's `on_stuck` reaches Webster-Bouncer, then Describe, then Publish.
  F1 and F6 are the defects.
- D4: every case is decided once a file's absence counts as content, including deletions.
  A task-side deletion differs from both points and is reported.
  A parent-side deletion the task synced equals the parent and drops out.
  One the task never synced equals the fork point and drops out.
- D6's claim that the orch is slug-free is consistent with today's code: every `lyx orch` verb refuses outside the prime, and a prime-parented task's parent name is formed slug-free by `hubgeom.decideParent`.
- Auto-mode assumptions: each names a self-pick that the Decisions carry; marking the picks for operator confirmation is the section's role, so they are not a relocation finding.
- Support log against decision record: no contradiction found.
  No requirement sits only in the support log, apart from the `orch-any-worktree` dependency in the Question ledger, which belongs to that board entry rather than to this plan.
