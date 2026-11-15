# Round 2 fixer-report

## Changes

- **F2** (fixed differently; commits `370f3b0e`, then the follow-up `orchengine: fix F2 — a failed end ends the watcher as done…`).
  - The review proposed: record the exit and return the end's error with `done=false`.
  - Not made: the next tick finds the strand gone and `watcher.go:277` overwrites `WatcherExit` with `strand gone`, so the reason would be lost again (`watchloop.go:206` also lets an error win over `done`).
  - Made instead: when `Stop` succeeded, `ownerTick` saves `owned by machine <owner>` and returns `done=true`;
    a failed end is logged at Warn and is not the tick's error, since it is never retried.
    A failed `Stop` still returns the joined error with `done=false` and records nothing.
  - Updated the `ownerTick` doc, the matching paragraph of `internal/orchengine/doc.go`, and the test row, renamed `FailingEndStillStopsTheStrandAndEndsTheWatcherWithTheOwnerExit`, to expect `wantDone` and `wantExit "owned by machine there"`.
- **F3** (fixed as proposed; commit `4bce3101`): added `# Ending a subject owned elsewhere` to `internal/pairteardown/doc.go`.

Gate: `lym gate test ./internal/orchengine ./internal/pairteardown` passed; `./internal/orchengine` passed again after the follow-up.

## Self-check

- fix-proposal, BLOCKING: the F2 exit is overwritten by the next tick — fixed differently, as above.
- fix-proposal, LOW: `owner.go` and `doc.go` disagree on `done` — fixed by the same change.
- fix-code-correctness: reported, no findings (its note that a later tick overwrites the exit is the same defect).
- fix-consistency: reported, no findings.
- fix-affected-set: reported, no findings.

## Disputed

None.

## Deferred

- **F1** (`_lym/landing/summary.md:34`): not done.
  `_lym` links to the `orch-resume-own-session-weft` worktree, outside this worktree, and the round's rules bar any git command on a `_lym` path.
  The parent, or whoever writes that worktree's `summary.md`, makes the rewrite the review's `fix` key describes.
  Note for that rewrite: the watcher now ends as done with the owner exit even when the end failed, logging the end failure at Warn instead of returning it.

## Rulings

None.
