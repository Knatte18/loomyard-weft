---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/clihelp/index_test.go:119
    summary: TestRenderIndex_FormsAndFiltering runs parallel subtests over one shared cobra tree, so lazy flag merging races and the Publish verify fails with concurrent map writes
---

Review range: the status file gives `product.parent` `main` and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, plus the merge-in `fc7afc15c` the Publish failure names.
The branch's own code in `internal/shuttleengine/send.go` (idle wait, paste path, pending-send retry, guarded Enter) matches the plan's guard and exact-match-Enter design, and I found no defect in it.

### [BLOCKING] Parallel subtests share one cobra command tree

**Location:** `internal/clihelp/index_test.go:119` (the `t.Run` loop at the end of `TestRenderIndex_FormsAndFiltering`), failing through `internal/clihelp/index.go:94` (`visibleLocalFlags`).
**Issue:** `.lyx/verify/verify-4.log` shows `fatal error: concurrent map writes` in `internal/clihelp`, raised from `TestRenderIndex_FormsAndFiltering.func1`.
Every subtest calls `t.Parallel()` and then `RenderIndex(root, ...)` on the same `root`.
`visibleLocalFlags` calls `cmd.LocalNonPersistentFlags()`, which in cobra merges parent persistent flags into the command's flag set on first use and writes `pflag.FlagSet`'s maps.
Three goroutines doing that on the same commands is a data race, so the test fails nondeterministically.
The branch does not touch `internal/clihelp` (`git diff main..HEAD -- internal/clihelp` is empty) and the merge-in `fc7afc15c` brought in no change there either, so the fault predates this branch.
It still fails the Publish verify that this round exists to fix, and `lyx gate test ./internal/clihelp` passing from cache does not clear a race that depends on scheduling.
**Fix:** Make the subtests not share mutable cobra state: drop `t.Parallel()` from the subtests, or build the fixture tree inside each subtest, or call `RenderIndex` once per audience before the parallel phase so the lazy flag merge has already happened.
The first is the smallest change; keep the table as it is.
