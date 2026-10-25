# Fixer-report — code round 1

## Changes

- **F1 (BLOCKING), fixed as proposed**: `worktreeHasDobby` in `internal/battencli/overview.go` attaches the dobby section only for the prime or a beam from `hubgeom.DobbyStart(location).Beams`; a task pair logs a Debug line and draws none. `TestWorktreeHasDobby` has rows for the prime, a beam, a task pair and an unreadable beam list.
- **F2 (MEDIUM), fixed with my own change**: the proposal was a non-acquiring probe or a comment change plus a check that the acquirers tolerate a refusal.
  No portable read-only lock probe exists (flock offers only try-acquire), and the acquirers did not tolerate a refusal.
  `DriveVerb` and `dobbyengine.Run` now wait `lock.ProbeGrace` (new constant, 250ms) through `AcquireWriteLockWithin` before refusing.
  The comments in `cli.go`, `dobbyreads.go`, `overview.go`, `sheddriver/drive.go`, `dobbyengine/loop.go` and `doc.go` say the pane takes the locks for an instant.
  `TestDriveVerb` has a row for a lock released within the grace.
- **F3 (MEDIUM), fixed as proposed**: `TestDobbyTasks_ReadsDependenciesAndParentOfAReleasedTask` in `dobbyreading_test.go` (tier0, temp board dir).
- **F4 (MEDIUM), fixed as proposed**: a subtest in `TestStartReady_RefusalsAndBackoff` asserts the refusal text is written, then that the start record is gone once a retry's pass leaves a status file.
- **F5 (LOW), fixed as proposed**: `BuildView` counts a done run holding its driver lock in the slots, and keeps it out of `view.Runs`; a new `TestBuildView` row pins it.
  A comment names the residual: the view cannot tell the dobby's own pass from a hand-run driver.
- **F6 (LOW), fixed as proposed**: `TestLog_OneLinePerStartAndPass` has a run with the origin-owed marker and expects `pass o full-no-reopen`.
- **F7 (NIT), fixed as proposed**: `overviewCmd`'s comment and `Short` name the runs strand of the prime and of each beam.

## Self-check

- fix-consistency, LOW, `docs/overview.md` "reads files only": fixed.
- fix-consistency, LOW, `internal/loomcli/doc.go` "refuses at once": fixed.
- fix-consistency, LOW, `ProbeGrace` and other try-acquirers: fixed as proposed, no change; the other try-acquirers take the run, land or other locks, which the pane never probes.
- fix-affected-set, LOW, the same two stale wordings: fixed with the two above (the `TestDriveVerb` comment included).
- fix-proposal, MEDIUM, F5 view and scheduler can disagree for a hand-run driver: fixed with my own change, a comment naming the residual; no outside reader can tell the dobby's pass from a hand-run driver, so the proposed proxy is the closest available.
- fix-proposal, LOW, probe collisions still possible: fixed as proposed, no change; transient, and the grace covers the acquirers the review named.
- fix-proposal, LOW, `localBeams` failing on an unrelated pair: fixed as proposed, no change; the Warn names it, and the failure shows no section rather than a wrong one.
- fix-code-correctness, LOW, stale loomcli wording: duplicate of fix-consistency.
- fix-code-correctness, LOW, F5 residual: duplicate of fix-proposal.
- fix-code-correctness, LOW, F2 has no test: fixed, the `TestDriveVerb` row.

## Disputed

None.

## Deferred

None.
