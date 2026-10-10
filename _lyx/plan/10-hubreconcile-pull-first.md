# Card 10 — hubreconcile-pull-first

**Edit:**
- `internal/hubreconcile#ensure`
- `internal/hubreconcile#walkHub`
- `internal/hubreconcile#reconcileHubWide` -> `func reconcileHubWide(boardDir, primeAnchor, label string) (skipped bool, err error)`
- `internal/hubreconcile/ensure_integration_test.go`
- `internal/hubreconcile/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/hubreconcile#revisionLabel`
- `internal/hubreconcile#shortRevisionLength`

**Uses:**
- `plan:internal/buildvcs#Identity.Label`
- `plan:internal/fabricengine#BinaryLabel`
- `plan:internal/fabricengine#Bolt.PullThenCommitWritten`
- `plan:internal/fabricengine#BoltWriteResult`

**Intent:** D3 and D4 for the hub-wide config write.
`ensure` reads the per-worktree label from `running.Identity.Label()`, the moved copy of `revisionLabel`, and `walkHub` passes `fabricengine.BinaryLabel()` to `reconcileHubWide`, whose commit subject becomes `lyx: reconcile hub-wide config for build <label>` with the shared parenthesised label; the per-worktree commits keep their `for build <short revision>` subject.
`reconcileHubWide` commits through `Bolt.PullThenCommitWritten`, restoring the config snapshot on a write or commit error as today; a skipped result, fetch failure or divergence, leaves the files unwritten and reports `skipped`, and a push failure after a landed commit logs the Warn as today and also reports `skipped`.
`walkHub` folds that into the skipped it already returns for a mid-merge worktree, so `ensure` leaves the stamp absent and the next start verb reruns the walk; a rerun whose unpushed hub-wide commit sits over a moved upstream meets the divergence skip, keeps the stamp absent and repeats the Warn, and only a resolved `git pull --rebase` in the board dir, or a board push that lands the commit after a transport failure, clears it.
`revisionLabel` and `shortRevisionLength` are deleted with their last caller moved.
`doc.go` adds the pull-first step and the stamp-absent rule for a skipped or unpushed hub-wide commit beside the mid-merge one.
Tests: `TestEnsure_UnreachableBoardRemoteStillCommitsHubWideConfig` becomes `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent`, asserting nothing is written, the fetch Warn is logged and no stamp exists; `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` asserts the hub-wide subject carries the label; a new `TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent` lands the hub-wide commit with the push rejected by a commit pushed from a second clone, then runs `Ensure` again and asserts the divergence Warn naming `git pull --rebase`, no new commit and no stamp; a new row in the stale-build test seeds the board clone behind its upstream and asserts the hub-wide commit lands on top of the pulled copy; the budget row for `internal/hubreconcile` rises by one.

**ImpactSummary:** Re-signs one private function whose only caller is on this card and deletes two private symbols whose only caller is edited here; the observable change is the hub-wide commit subject and a stamp left absent when Bolt could not be brought up to date.
