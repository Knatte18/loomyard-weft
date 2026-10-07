---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-gitrepo-leased-update.md, _lyx/plan/11-reset-moves-remote-branch.md
    summary: The leased remote update returns an unclassified error, so reset cannot tell a lost lease from a network or authentication failure, though D7 gives them different ways forward
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-gitrepo-leased-update.md
    summary: cmd/lyx/gitrepoboundary_test.go pins every gitrepo method that calls runChecked, so the two new methods fail go test until that list names them, and no card edits it
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/11-reset-moves-remote-branch.md
    summary: The remote update is a force push inside fabric but the card names no gated executor and no destructive-guard entry in cmd/lyx/destructiveguard_test.go
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-uses-later-target.md
    summary: Two Run tests in websterengine/runlevel_test.go build a backward Uses plan that the new check refuses at run entry, and card 1 leaves them to an unnamed grep
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/08-background-shell-outcome.md
    summary: The outcome list omits paused, which mapMasterDone returns as a RunResult outcome
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/06-rebaseline-terminal-batch.md
    summary: The done row of the planned table repeats two existing tests, and the reworded done refusal is not tied to their existing substring assertion
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/11-reset-moves-remote-branch.md
    summary: Two planned reset test rows repeat existing tests, which should be extended instead
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/03-delete-target-gone.md
    summary: The whole-plan row for a missing Delete target extends the scope axis that TestValidateCmd_ScopeFollowsRunProgress already covers
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/02-delete-before-reference.md
    summary: The decision record says new plan checks get rows in the spec's Validation checks list, but cards 2 and 3 document their checks in a paragraph with no row
---

### [BLOCKING] Leased update has no lease-lost discriminator

**Location:** `_lyx/plan/09-gitrepo-leased-update.md` (`UpdateRemoteBranchLeased`), `_lyx/plan/11-reset-moves-remote-branch.md` (refusals).
**Issue:**
D7 gives two different refusals.
A remote tip that moved after the fetch refuses with the `ours`-merge way forward.
A fetch or update that fails for another reason refuses with "re-run once the remote is reachable".
Card 9 specifies `UpdateRemoteBranchLeased` as returning a wrapped `*gitexec.GitError`, the shape `DeleteRemoteBranchLeased` has, with no sentinel or other marker for a rejected lease.
Card 11 says "the lease turns [a moved tip] into the same refusal", but nothing in either card lets `ResetPairWarp` tell the two apart.
An implementer must either string-match git's stderr in `fabricengine` (which `PATTERN-fabric-git` and `PATTERN-gitrepo-client-boundary` keep inside `gitrepo`) or send both cases to one way forward, which is wrong for at least one of them.
Card 11's tests also never reach the second branch: the unreachable-remote row fails in the fetch, so an update that fails for a non-lease reason has no test, though `PATTERN-refusal-way-forward` requires a test that reaches each refusal.
**Fix:**
Card 9 adds a sentinel for a rejected lease (`ErrLeaseRejected`, returned bare so `errors.Is` works, classified from git's stale-info stderr in `gitrepo` the way `ErrPushRejected` is classified from `rebaseRetryTriggers`) and a push_test.go assertion on it.
Card 11 maps the sentinel to the divergence refusal and every other update error to the "re-run once reachable" refusal, and adds a test row where the update fails for a non-lease reason (a `pre-receive` hook on the hubforge origin that exits non-zero), asserting the refusal and that remote and local are unchanged.

### [MEDIUM] Boundary test pins the runChecked method set

**Location:** `_lyx/plan/09-gitrepo-leased-update.md`.
**Issue:**
`cmd/lyx/gitrepoboundary_test.go` holds `gitrepoPinnedRunBoundMethods` and fails when the set of `*Repo` methods whose body calls `r.run` or `r.runChecked` differs from it.
Card 9 adds `UpdateRemoteBranchLeased` and `CommitsNotIn`, both through `runChecked`, and the plan's `go test ./...` fails at that card until both names are added.
No card names the file, and the card's own boundary sentence cites `PATTERN-gitrepo-client-boundary` without it.
**Fix:**
Add `cmd/lyx/gitrepoboundary_test.go` to card 9's `**Edit:**` list, state in its Intent that both method names join `gitrepoPinnedRunBoundMethods`, and add `./cmd/lyx/...` to the plan's `-tags integration` verify line, since `cmd/lyx` carries integration tests.

### [MEDIUM] Remote force update has no gated executor or guard entry

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md`.
**Issue:**
`PATTERN-fabric-destruction-chokepoint` makes `destroy.go` the only file with a destructive primitive and requires every executor to check containment, ownership, dirtiness and force in order, with `rec` threaded in.
D7 names the remote update as a new lease-guarded update in `destroy.go`.
The card describes steps inside `ResetPairWarp` but never says the update is an executor, how it sits relative to the gate that `resetHardTo` runs together with the reset (the card needs the gate before the fetch and the update, and the reset after), or how `cmd/lyx/destructiveguard_test.go` keeps another fabric file from calling `UpdateRemoteBranchLeased`.
That guard bans `.DeleteRemoteBranch(` outside `destroy.go` for the same reason and pins every executor in `destructiveGuardRecordingExecutors`, so a new force push without an entry is the exact gap the guard exists to close.
The card also edits `destroy.go`'s file comment, which pins a tally ("every one of the nine executors").
**Fix:**
State in the Intent that the remote update is a new executor in `destroy.go` taking a leading `rec *Mutations` and running the existing gate before it pushes, with `ResetPairWarp` calling the gate, then the fetch and ancestry read, then the executor, then `resetHardTo`.
Add `cmd/lyx/destructiveguard_test.go` to the Edit list: a `.UpdateRemoteBranchLeased(` banned token and a `destructiveGuardRecordingExecutors` row for the executor.
Reword the `destroy.go` comment so it counts no executors.

### [MEDIUM] Run tests that build a backward Uses plan are not named

**Location:** `_lyx/plan/01-uses-later-target.md`.
**Issue:**
`TestRun_ReorderingIsObservableInMasterPrompt` and `TestRun_CycleReportingSurfacesOnRunResultButNeverFails` in `internal/websterengine/runlevel_test.go` call `addCardUses` so card 1 `Uses:` card 2's Create target (the cycle test also makes card 2 use card 1's).
Run entry validation runs the format set, so after this card both tests are refused before Master spawns and fail.
Their scenario, a plan Webster must reorder, is exactly the shape D1 keeps out of the Plan-Gate.
Card 1 names `handlePlan` in `rebaseline_test.go` and leaves "any further" cases to a grep, and `runlevel_test.go` is not in its Edit list (cards 4 and 8 edit it later, for other reasons).
**Fix:**
Add `internal/websterengine/runlevel_test.go` to card 1's Edit list and say the two tests are deleted: the backward-`Uses` plan cannot reach `Run` any more, and reordering and cycle condensation stay covered by `sequence_test.go`'s tests of `SequenceBatches`.
`TestRun_AcyclicPlanReportsNoCycles` stays.

### [LOW] Outcome list leaves out paused

**Location:** `_lyx/plan/08-background-shell-outcome.md`.
**Issue:**
D9 and the card list the outcome as done, stuck with its reason, or the error.
`mapMasterDone` also returns `outcomePaused` in `RunResult.Outcome` (it skips `ClearPause` only for it), and the card moves the warnings to "every outcome that returns a `RunResult`" while naming only done and stuck.
A paused run with an expired shell would get wording for no outcome.
**Fix:**
Name paused in the outcome description (shuttle-done, so Master's strand is removed like the other `RunResult` outcomes), and add a paused row to `triage_test.go`'s outcome table.

### [LOW] Done row repeats existing rebaseline tests

**Location:** `_lyx/plan/06-rebaseline-terminal-batch.md`.
**Issue:**
The planned table has a `done` row refusing an edited named card.
`TestRebaseline_RefusesChangedBegunCardBody` and `TestRebaseline_ForeignEditToBegunCardStaysRefused` already drive exactly that (a done batch one, `--card 1`).
The card also rewords the done refusal to say why `--card` cannot accept it, while the first test asserts the substring `batch 1 card 01-json-flag changed since it was begun`; the card does not say the new text keeps it.
**Fix:**
Drop the `done` row from the new table, extend `TestRebaseline_RefusesChangedBegunCardBody` to assert the new done-batch reason, and state that the reworded message keeps the `changed since it was begun` text.

### [LOW] Two reset test rows repeat existing tests

**Location:** `_lyx/plan/11-reset-moves-remote-branch.md`.
**Issue:**
The row "a dirty tracked path outside `ownPaths` refuses before any fetch, remote untouched" is `TestResetPairWarp_DirtyPathOutsideOwnPathsRefuses` plus one assertion.
The row "after pushed commits the reset leaves the remote branch at `sha`" is `TestResetPairWarp_DiscardsCommitsAndOwnPathDirtKeepsUntracked` plus a remote assertion on a hub with an origin.
**Fix:**
Say the card extends those two tests with the remote assertions (and pushes before the reset in the second), and keeps new functions for the cases no existing test covers.

### [LOW] Whole-plan Delete row repeats the scope test

**Location:** `_lyx/plan/03-delete-target-gone.md`.
**Issue:**
`TestValidateCmd_ScopeFollowsRunProgress` already checks that `lyx webster validate` gives the whole-plan answer with no run recorded and the pending-scope answer once a batch is begun, which is the same axis as D3's "keeps refusing before the run starts".
**Fix:**
Cover the no-run-state refusal and the begun-batch advisory by adding a Delete card to that test's fixture or one more sub-test of it, instead of a row in the new `TestValidateCmd_PlanGateChecks` table.

### [LOW] New check IDs get no row in the Validation checks list

**Location:** `_lyx/plan/02-delete-before-reference.md`, `_lyx/plan/03-delete-target-gone.md`.
**Issue:**
The decision record's constraint says new plan checks get rows in `loom-plan-spec.md`'s "Validation checks" list and in the owning package's `doc.go`.
Card 1 does both.
Cards 2 and 3 add `doc.go` entries and a paragraph after the list, so the list, which calls itself the authoritative one-row-per-ID list, omits two IDs the same entry points emit.
**Fix:**
Add rows for `delete-before-reference` and `delete-target-gone` to the list, after the rows card 1 numbers, saying they are emitted by `internal/planglyph` rather than `internal/planparser`; keep the paragraph for the rule detail.
