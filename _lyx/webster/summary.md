# batten-cancel-verb: `lyx batten cancel <slug> [--remove]`

All four batches recorded `status: done`; no batch needed recovery, and no digest carried a departure or a warning.

## Batches

- **01-cancel-marker-paths** (head `5e321cf2`): the cancel marker's path and lock path in shedrun, a process's creation time by pid on Windows, the pair add's typed dirty-source refusal and probe, fabric's local pair-parts read and origin task-branch read, and fabric's archive-and-delete of the task branch locally and on origin.
  Scope guard deviations: `internal/fabricengine/archive.go`, `destroy.go`, `export_test.go`, `pairparts.go`, `taskbranchdiscard.go` and `taskbranchdiscard_integration_test.go`.
- **06-records-write-lock** (head `ceb5c53c`): the exported combined records write lock hold, shed's precondition prefix and the check `Run` and `Step` call under the run lock, the driver's precondition halt, Worktree-Create's halt on a `PreconditionError`, batten's mapping of a dirty source and a left task branch, and the board's run-lock refusal naming the cancel.
  Scope guard deviations: `internal/battencli/lifecycle_integration_test.go` and `internal/shedengine/run.go`.
- **12-cleanup-removal-split** (head `d8e7a6e3`): the per-slug removal split from batten cleanup's locks and judgement, the origin-mode snapshot and claim-leaving removal, pairteardown's cancel phase, the cancel marker's write, read and liveness lock, the pre-run's marker, dirty-source and left-parts refusals, and the marker and seed re-read in the seeding closure and under shed's run lock.
  Scope guard deviations: `internal/battencli/arm.go`, `cancelmarker.go`, `cancelmarker_test.go`, `cleanup.go`, `predrive_test.go`, `startguard.go`, `startguard_integration_test.go`, `internal/pairteardown/cancel.go` and `cancel_test.go`.
- **18-batten-cancel-core** (head `c30e292d`): the cancel's decisions and order over injected seams, the verb with its production seams and judge-only reconcile, and the overview and orch stencil naming the verb.
  Scope guard deviations: `internal/battencli/cancel.go`, `cancel_integration_test.go`, `cancel_test.go` and `cancelcmd.go`.

## Departures

None: no digest listed a `Pattern-Departure:` trailer.
The declared `PATTERN-pair-teardown` change is carried by the cancel phase in `internal/pairteardown`.

## Verify gate findings

The plan-level verify failed on attempt 1 of 3 in `internal/reedcli` (`TestStrandNamingAndList/HubWideDirectoryAndDrift`, `TestNaming/TaskWorktreeStrandNameTitleAndEnv`, `TestNaming/EmptySlugGivesShortnameAndRole`, `TestNaming/RepairNamesRestoresAHandChangedTitle`, `TestOwnWindow_RealTmux/ShellOnlyStrandReadsDeadPastTheGraceAndResumeRelaunchesIt`) and in the `internal/reedengine` package, whose FAIL line had no failing test under it.
The findings file noted that no recorded card touched a failing package.
Attempt 2 failed on the same tests: tmux 3.4 prints the `\x1f` pane-tail separator as the literal text `\037`, a defect on main that this branch was cut before.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `0e0c9df2eee143470c8a5a32bf1aeb92aed6abe3` fix: reedengine pane-tail separator is a tab, which tmux 3.4 passes through verbatim
