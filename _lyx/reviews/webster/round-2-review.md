---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/smoke_test.go:616
    summary: origin record self-heal smoke step runs git rm in the records worktree while an earlier step's driver is still alive, and fails on a weft index.lock collision
    fix: In the subtest "origin record self-heals after a crash between write and commit", call killDriversAndClearLog(t) as the first statement, before the gitkit.Git "rm" call on recordsDir. killDriversAndClearLog kills the Go driver and waits for the run lock to free, so no driver is left to take the records worktree's index.lock while the step edits its index.
---

### [BLOCKING] Origin-record self-heal smoke step races a live driver for the weft index lock

**Location:** `internal/loomcli/smoke_test.go:616` (the `gitkit.Git(t, recordsDir, "rm", ...)` call in the subtest "origin record self-heals after a crash between write and commit"); failing run `.lyx/verify/verify-7.log`.

**Issue:** The Publish verify failed on `TestSmokeBootstrapLifecycle/origin_record_self-heals_after_a_crash_between_write_and_commit`.
The log shows `git rm -q -- _lyx/fabric/origin.json` exiting 128 with `Unable to create '.../warp-bare-weft/.git/worktrees/loom-smoke-task-weft/index.lock': File exists`, so another git process held the records worktree's index at that moment.
The preceding subtest, "second invocation does not spawn a second driver", deliberately leaves the first bootstrap's driver running, and the failing subtest never stops it.
A live driver commits status into the same records worktree, so the test's own `git rm` and `git commit` collide with it whenever a driver transition lands in that window.
The subtest that follows ("died driver proceeds ...") already calls `killDriversAndClearLog` before it touches the records worktree; this one does not, which makes the failure timing-dependent rather than deterministic.

The test file is not changed by the branch's own range, and the branch's non-test changes (the `validate-discussion` darn path, `wireLightweight` seams, `landingDeps` signature) run only on a darn run or at landing, not in this loom-recipe smoke driver.
I found no branch change that adds a git call on the records worktree in this path.
The smoke suite is tmux-tier and cannot be run in this session, so the diagnosis rests on the log and on reading the test; it is not reproduced.
The merge-in diff (`cf2ef642c^1..cf2ef642c`) touches `shedrecipe` discussion entries, `loomengine` config and `shuttleengine` sends, which may have shifted driver timing, but I could not tie the collision to one of them.
The fix removes the dependence on timing whichever change exposed it.

## Scope note

The branch's own range (`git diff $(git merge-base main HEAD)..HEAD`, seventy files) was sampled, not exhaustively re-derived: the non-test `loomcli`, `fabricengine/committedfile.go` and `envkit` changes were read against the plan's cards and showed no defect.
Round 1 converged on the rest, and the focus directive set no further focus.
