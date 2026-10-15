## Changes

- F1 (LOW): fixed as proposed, by the review's alternative.
  `RunTally.WebsterEngine`'s comment in `tools/tokencount/count.go` and the calibration paragraph in `tools/tokencount/main.go` now state that calibration assumes every webster-role session of a run ran under one engine.
  Committed as `ebc6a060c`; the change is comments only.
  The primary alternative, preferring the recorded `masterEngine`, would need run records plumbed into `CountRun`, which has none today, so the documented assumption was chosen.
- Gate finding (uncommitted changes blocked the `tokencount` verify): my `go build ./tools/tokencount/` had left an untracked `tokencount` binary in the worktree root.
  Removed it; `git status` is clean.

## Disputed

None.

## Deferred

None.
