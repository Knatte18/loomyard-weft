# Fixer-report, round 1

## Changes

- **F1** (fixed as proposed, plus one adjustment): `presentTaskWorktreePath` in `internal/battencli/wire.go` now asks `taskWorktreePresent` and returns the by-name refusal, now built by the new `absentTaskWorktreeRefusal`, for a stray directory too. So `taskWorktreeLocation`, goto and the readopt `SpawnPush` seam refuse it by name.
  - Adjustment: the cached branch of `taskWorktreeLocator` keeps its stat-only check and uses `absentTaskWorktreeRefusal` directly.
  - Why: `stepTaskWorktreeSeams_RunGitOnlyUntilTheFirstResolution` hides `.git` after the first resolution and requires the cached answer without git, and the first gate run failed on it.
  - Commit `ec00d0ed`.
- **F2** (fixed as proposed): subtest `MarkedPairOfATaskWhoseWorkReachedItsParentPasses` in `TestBeamLanding_ReachesWorktreeCreateSkip` drives `hubStartGuard(h.Location, "promoted")` with batten at Worktree-Teardown and asserts a nil error. Dropping the `ReachedParent` wiring fails it. Commit `8ed64560`.
- **Gate repair:** the `taskWorktreeLocation` doc sentence I added in `wire.go` was a fixed-column wrap; it now breaks at the sentence boundary, one sentence per line.
- **Self-check fixes** (docs only, commit `bb5f99a6`): the `cli.go` sentence on a gone task worktree and the `taskWorktreePresent` and `taskWorktreeLocation` docs in `wire.go` now name the verbs that gate on or resolve the task worktree instead of claiming every verb, and the locator doc says later calls stat without spawning git.

The gate (`lym gate test ./internal/battencli`, tiers 0 and 1) passed after the code changes.

## Self-check

- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-consistency, LOW, `cli.go:69` and the `taskWorktreePresent` doc overclaim "every batten verb" and "one answer" while `pairPresent` (`beamland.go:140`) and the overview (`overview.go:290`) only stat: fixed with my own change. I softened the docs rather than routing the two listing sites through `taskWorktreePresent`, which no finding asked for and which changes what the overview and beam-land member listing show.
- fix-consistency, LOW, `taskWorktreeLocation` doc silent about a non-resolving directory: fixed as proposed.
- fix-affected-set, LOW, the locator's cached branch is stat-only so "every verb" does not hold there: fixed by the same doc change, which names the cached branch's stat-only behavior in the locator doc and drops the "every verb" claim.
- fix-affected-set, LOW, other stat-only sites (`beamland.go:140`, `overview.go:290`, `refusal.go:122`): same disposition as the consistency finding on them; the docs now say the overview and member listing only look for the directory.

## Disputed

None.

## Deferred

None.
