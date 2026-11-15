# Fixer-report, code review round 1

All 13 findings are fixed; none is disputed.
The packages `runproc`, `orchengine`, `pairteardown`, `orchcli` and `shuttleengine` pass `lym gate test` at tier 0 (2200 passed, 0 failed), and `go vet` is clean over them.
The commits group findings by the files they share, since F2 to F6 and F12 change one signature and its callers together.

## Changes

- **F1** (commit `pairteardown: fix F1`): `endOwnedProcesses` in `internal/pairteardown/teardown.go` now always spares `runproc.RoleOrch` beside `RoleTeardown`.
  The `endOwned` field comment, `internal/pairteardown/doc.go` step 3 and its session-end sentence say so.
  `internal/beamcli/doc.go`'s Bound names no orch, so it stays.
  The `summary.md` bullet the review names is in the `orch-resume-own-session-weft` worktree under `_lym/landing`, outside this worktree and write surface, so it is not edited (see Deferred).
- **F2**: `runproc.endSubject` spares `RoleTeardown` in the runs pass.
  The subject pass spares `RoleDobby`, `RoleTeardown`, `RoleAdopt`, `RoleSync`, `RolePush`, `RoleTestMap` and `RoleSlackRelay`.
  `EndSubject`'s comment and `internal/runproc/doc.go` name them.
  `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` gains a teardown record under a run and under the beam, and a `HubOwner` row with push, test-map, slack-relay and teardown records that asserts none is signalled.
- **F3**: `EndSubject` and `ender.endSubject` take `endDobby bool`, and the dobby pass runs only when it is set.
  The watcher's wiring in `internal/orchcli/cli.go` passes false, so the dobby's own hand-over pauses the runs and archives.
  A new row (`AnUnsetEndDobbyLeavesTheSubjectsDobbyRunning`) pins that a live dobby is not signalled.
  The `OwnerWatch.End` comment, the orchengine and orchcli docs and `PATTERN-child-processes.md` follow.
- **F4**: fixed with my own change to the location.
  The review proposed `tendercli`, but `tendercli` imports `orchcli` (see `internal/orchcli/doc.go`, "`tendercli` imports this package"), so `orchcli`'s wiring could not call it without an import cycle.
  The selection moved to `pairteardown.EndSubjectProcesses(prime, owner, endDobby)` in the new `internal/pairteardown/endsubject.go`.
  `tendercli` and `hibernatecli` import `pairteardown` already, and `orchcli` does too.
  `endSubjectProcesses` is deleted from `internal/orchcli/watch.go`, and `c.endSubject` calls the new function.
  The `orchcli` doc names it.
- **F5**: the selection is a pure function, `subjectRuns`, in `endsubject.go`.
  `TestSubjectRuns` in `internal/pairteardown/endsubject_test.go` pins a beam (its tasks only, never its own record, never an unreadable owner), another beam, and the root (root tasks only).
  The proposed seam on the final `EndSubject` call is not added: the runs handed to it are exactly `subjectRuns`' result, so testing it directly covers the selection without a second fake.
- **F6**: `pattern/PATTERN-child-processes.md` Enforcement now lists `EndSubject`.
- **F7**: `ownerTick` in `internal/orchengine/owner.go` calls `Stop` even when `End` failed and returns the joined errors with `done` false.
  The `FailingEndReturnsTheErrorAndStopsNothing` row becomes `FailingEndStillStopsTheStrandAndReturnsTheError` (calls end then stop s1), and a row for both failing is added.
  The review's `fix` says the next tick retries the end; it does not, because the next tick finds the strand gone and the watcher exits.
  The `ownerTick` and orchengine docs say that instead (see Deferred).
- **F8, F9**: two rows in `TestStart_ResumeOverALiveHolder`: a failing `orchRegistered` read (new `registeredErr` field), and `resumeErrs {held, held}` with the own holder registered (one end, two starts, an error naming `lym beam start`).
- **F10**: `fakeStarter` gains `holderErr`, and `TestStart_RegistersTheLaunchedOrchProcess` gains a row asserting the warning carries its text.
  The new `internal/shuttleengine/run_sessionholder_test.go` tests `Run.SessionHolder` over an engine that names a holder, one that finds none, one whose read fails, one without the capability and a run with no session id, and `Run.CarriedInstructions`.
- **F11**: the `//testtiming:keep` line above `TestStart_StarterErrorFailsAndSavesNoState`.
- **F12**: `internal/runproc/doc.go` names `RoleOrch` in the `HubOwner` role list.
- **F13**: the failed-resume row's error is `pane never came up`.
- Gate repair: the comment lines this round added that reached 120 characters or wrapped mid-sentence in `internal/orchengine/doc.go`, `internal/pairteardown/doc.go`, `internal/pairteardown/teardown.go`, `internal/runproc/doc.go` and `internal/runproc/end.go` are split one sentence per line; no wording or behaviour changed.
- Follow-up from the self-check: the session-end sentence in `internal/pairteardown/doc.go` also names the orch and teardown children as always spared.

## Disputed

None.

## Deferred

- **summary.md** (F1 and the design-decision bullet, F4's "update summary.md to name that function"): the file is `/home/knatte/Code/loomyard-LYMHUB/orch-resume-own-session-weft/_lym/landing/summary.md`, in another worktree under `_lym`, which this round must not edit.
  The parent should correct the bullet that says a pair teardown ending the orch matches D9, and name `pairteardown.EndSubjectProcesses` as the function the dobby's hand-over and the catch-up call.
- **F7's retry**: with `Stop` called regardless, a transient failure of the end, such as an unreadable board, is not retried, because the watcher exits on its gone strand;
  keeping the strand up for a retry is the behaviour F7 reports as the defect.
  Whether to stop only for a persistent failure such as a `SurvivorsError` is an operator tradeoff, not decided here.
  The fork's finding below is the same point.

## Rulings

None reached me.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: the session-end sentence in `pairteardown/doc.go` names only the dobby as spared. Fixed as proposed.
- fix-affected-set, MEDIUM: `tendercli.EndOwnedElsewhere` ends the orch through `runproc.End`. Disputed, premise false: the function runs only for a slug another machine owns (`internal/tendercli/deps.go:285-289`, "because another machine owns the run"), which D9 licenses.
- fix-affected-set, LOW: the tender's `EndOwner` seam ends an owner with `runproc.End`. Disputed, premise false: it ends the processes of an owner the tender found orphaned or stalled, and ends every record by design; the fork did not show a path that reaches a beam this machine owns.
- fix-proposal, MEDIUM (F7): a failed end is never retried once the strand stops. Deferred, an operator tradeoff, as above.
- fix-proposal, LOW (F3): a live dobby could restart a run the watcher just ended. Disputed, premise false: the dobby reads the owner on each probe before its tick (`internal/dobbyengine/loop.go:550`), and `handOver` then pauses the runs it drives (`internal/dobbyengine/handover.go:69`), whatever the watcher ended.
- fix-proposal, LOW (F1): no end of the orch once the beam is torn down. Disputed, premise false: the teardown ends the pair's reed session (`internal/pairteardown/doc.go`, the sequence header), which holds the orch's pane and its process.
