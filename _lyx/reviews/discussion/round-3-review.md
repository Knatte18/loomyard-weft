---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D2 and D3 Bound
    summary: D2 never says what the row does when the stop of a live conflict-session strand fails, so D3's bound that a surviving session cannot write into the redone merge holds only on the success path
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 Decision and Rationale
    summary: D3 says resume's no-live-driver refusal names lyx loom start and that a dead driver is respawned only through start, but for a dead strand the refusal names lyx batten run then resume, with start only as the fallback
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D6 Decision
    summary: D6 does not state what the caller check does when hubgeom.ResolveParent returns an error, so the guard's behavior on an unreadable origin record is left to the plan writer
---

### [MEDIUM] D2 leaves a failed conflict-session stop undecided

**Location:** `_lyx/discussion/decision-record.md`, D2 Decision ("the row first stops any live conflict-session strand of its own run through shuttle's stop verb ... then aborts the merge") and D3 Bound ("a conflict session that outlived a dead driver is stopped by D2's row before the abort, so it cannot write into the redone merge").

**Issue:** The conflict session is its own shuttle strand that `mergeresolve` spawns through `Shuttle.Run` (`internal/mergeresolve/mergeresolve.go:99`), so it can outlive the driver that waited on it.
D2 orders stop, then abort, then a fresh merge-in, and D3's bound rests on the stop having worked.
Neither Decision says what the row does when the stop verb fails or the strand is still live afterwards.
If the row aborts and redoes anyway, the surviving session can write into the conflicted paths of the redone merge.
The resolver's re-verification only rescans conflict markers (`scanUnresolved`), so a stray write that leaves no markers passes it.
So the bound D3 states holds only when the stop succeeds, and the plan writer is left to choose between proceeding and stopping Stuck.

**Fix:** In D2, state that a stop that fails leaves the merge untouched and stops the row Stuck, with a reason that names the strand and its way forward.
Add that case to D2's Bound and to the Publish/Finalize acceptance criterion, and give its reason a refusal-spec row under `PATTERN-refusal-way-forward`.

### [LOW] D3 misstates resume's no-live-driver way forward and how a dead driver comes back

**Location:** `_lyx/discussion/decision-record.md`, D3 Decision ("so it reports its existing no-live-driver refusal, which names `lyx loom start`") and Rationale ("a dead driver is respawned only through `start`").

**Issue:** `noLiveDriverMessage` in `internal/loomcli/resume.go` gives three ways forward.
A retiring strand, or no strand, gets `lyx loom start`.
A dead strand gets `lyx batten run <slug>` from the prime, which revives the strand through reed (`battenshed.reviveDeadDriver`), and then `lyx loom resume` again.
`lyx loom start` is that message's fallback only.
So the Rationale rests on a false premise: a dead driver also comes back through batten's revive, and the hub's follow-up resume then meets D3's live-driver exception.
The acceptance criterion's wording, "the no-live-driver refusal", is accurate, but D3's text points a plan writer toward pinning `lyx loom start` as the way forward for every no-live-driver case.

**Fix:** In D3, call the refusal "its existing no-live-driver refusal and that refusal's way forward", without naming `lyx loom start`.
Rewrite the Rationale so a dead driver comes back through batten's revive or through `start`, and say the resume that follows either one passes under the live-driver exception.

### [LOW] D6 leaves the parent-resolution error path open

**Location:** `_lyx/discussion/decision-record.md`, D6 Decision.

**Issue:** D6 bases the check on the recorded parent name from `hubgeom.ResolveParent`.
That function returns an error when the origin record cannot be read or the prime cannot be told apart from a pair (`internal/hubgeom/parent.go`).
D6 never says what the verb does in that case.
Treating the error as "no parent", as `parentNameOrEmpty` does, refuses the real parent's own call and points it to the operator.
Refusing with the error stops every named caller and needs its own way forward.
An implementation that skips the check on error lets every task session through.
The guard's behavior on this path is a design call, and D6's Bound covers only a forged or cleared variable.

**Fix:** In D6, pick the behavior for an unresolvable parent, for example refusing every named caller with a way forward that names the unreadable record and says the operator's unnamed shell still passes.
Add that case to D6's Bound and to the `decision add` acceptance criterion.

## Focus directive verdicts

- D6 and a pair created from a task worktree: not a defect.
  `decideParent` forms `<shortname>:<parent-worktree>:orch`, and reed forms strand names through the same `agentname.Format`, so the recorded parent's own name passes the check, and the way forward never names a caller the check refuses.
  D6's Rationale states the slug-bearing parent, and Open risks states the `orch-any-worktree` dependency.
  The remaining gap in D6 is F3.
- Acceptance criteria and D3 against `resume.go` and `start.go`: what each verb proceeds over or refuses is attributed correctly.
  `start` reaches `refuseOverUnfinishedMerge` on both the spawn path and the parked-resume path.
  `resume` with a live, parked driver reaches the guard after the park-marker check.
  With no live driver, `resume` reaches the guard and then the no-live-driver refusal.
  The misattribution in D3's text is F2.
- D1 against `verifyGate.recordFailure`: not a defect.
  D1 states that a non-zero exit whose record write fails names no goto, and Scope adds the writer's report of whether it wrote.
  A written record with no parsed tests, for example a build failure, still carries the log path and the merge-in commit, so D1's "blind" premise does not cover it.
- D3 and a conflict session that outlived a dead driver: D2 states the stop, before the abort and through shuttle's stop verb, and both Bounds limit it to the run's own pair.
  The open path, a stop that fails, is F1.
