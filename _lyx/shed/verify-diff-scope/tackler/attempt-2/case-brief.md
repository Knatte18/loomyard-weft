# Case brief: Finalize stuck on "worktree dirty"

## 1. The question the next judge must answer

What dirtied the warp checkout `verify-diff-scope` during the Finalize merge check, and is it safe to re-run Finalize without a code change?

## 2. The facts

- The stop envelope `_lyx/shed/verify-diff-scope/drive-reports/drive-envelope-20261020-182433-e018.json` reports producer `Finalize`, state `blocked`, reason `fabricengine: merge preconditions failed: worktree dirty`, history length 35.
- The stop's trace `.lyx/shed/verify-diff-scope/steps/a13c141ec0058b65.trace.log` lines 12 to 14 show that the post-merge verify was skipped (tree already verified, tree `4c85f001`), then `landingshed: producer stuck` at 18:24:33.681 with the same reason.
- An earlier stuck with the same reason is in `.lyx/logs/trace-20261020T162352Z-739ccb768f9cb399-1209886.log` line 10 (18:23:55.682, trace `739ccb768f9cb399`).
- The halt note `_lyx/loom/friction/loom-halt-2.md` history shows `Finalize / stuck` at 16:23:55Z, `Finalize / goto` at 16:24:29Z, and `Finalize / stuck` at 16:24:33Z.
  So a `goto` resume already ran and returned the same stop unchanged.
- The guard is `pairDirtyReason` in `internal/fabricengine/mergeguards.go` lines 109 to 119.
  It runs `worktreeDirty(scopeTracked, f.warpPath)`, which is `git status --porcelain --untracked-files=no` in `internal/fabricengine/dirtiness.go` lines 57 to 70.
  Only the warp checkout is checked; the weft side is excluded by design (comment at `mergeguards.go` lines 104 to 108).
- The tackler's snapshot at spawn (`tackler/attempt-2/attempt.json`, `snapshot.dirty`) is `{}`, and `git status` in the warp checkout now prints nothing, untracked files included.
  HEAD is `b288ee68`, branch `verify-diff-scope`.
- The weft checkout `verify-diff-scope-weft` has only untracked `_lyx/` files.
  It is outside the guard's scope.

## 3. What I ruled out, and how

- Current dirtiness in the warp checkout: ruled out by `git status --porcelain` in the warp checkout (empty).
  The dirt is gone now, so the cause cannot be read from the working tree.
- Weft dirtiness: ruled out as the cause, because `pairDirtyReason` never reads the weft path.
- A stale reading in the spawn snapshot: ruled out, because the snapshot was also clean at 18:24:33.868.
- Tracked `_lyx/` or `.lyx/` files in the warp checkout: ruled out, because `git ls-files _lyx .lyx` prints nothing.
- Lock or git failure: not seen, because a git error would surface as a check error, not as `worktree dirty`.
- Not checked: whether the index was stat-dirty at the moment of the check; I did not run any git write command, and cannot tell from the logs.

## 4. Hypothesis (unverified)

A tracked file in the warp checkout was modified for a short window during the Finalize step, by a process I cannot see from the logs, and was restored or committed before this read.
If that holds, re-running Finalize would likely pass, but the second `resume` on this stop is not permitted once a resumed stop returns unchanged, and the run has no parent to hand the scope question to.
A stronger judge with the process and file history should confirm the cause before any re-launch.
