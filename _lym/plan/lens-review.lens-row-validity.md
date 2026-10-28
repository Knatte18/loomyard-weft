```yaml
lens: row-validity
outcome: reported
```

Lens report (row-validity): each planned test row checked for whether it reaches its branch, fails if the code were wrong, and is needed.

1. MEDIUM — Card 1, `TestFinalize_MergeInRequiredRetry` new row (AC: the retry after a merge-in-required error merges, gates and lands a code-side parent commit).
   The row claims each pass reads an overlapping parent change, but on the retry pass the second pass reads none.
   The first pass of `Finalize.mergeParent` rewrites the merge record with `MergedTip` = the local tip `p2`.
   On the retry, `Finalize.mergedTipRead` returns `p2`, which equals `newTip`, so `Finalize.judgeParentMerge` returns early with no overlap (`internal/landingshed/finalize.go:996`, `parentFrom == newTip`).
   So the retry pass never meets a large merge, and the row repeats the existing "retry lands" row's behaviour on that pass.
   To reach the branch, `ReadParentTips` has to answer a further moved tip (say `p3`) from the second merge on, standing in for a code-side commit that arrived between the passes.
   The card should also assert that `ChangedPaths` is called with `p2..p3` on the retry.

2. MEDIUM — Card 1, the same row, and the new `TestFinalize_GateAndFixer` large-merge subtest.
   Both replace `ReadParentTips` with "a local tip `p2`" and name no `Relation`.
   `newTestDeps`' default answers `Relation: fabricengine.ParentEqual` (`internal/landingshed/publish_test.go:70-72`).
   `Finalize.syncParent` calls the same seam with `fetch=true` and returns an error on any relation outside its switch (`internal/landingshed/finalize.go:393-402`).
   So a replacement with `Relation` left zero makes `Call` error before any merge, and neither row reaches its branch.
   Both cards should state `Relation: ParentEqual`, with `Upstream` equal to `Local`.

3. LOW — Card 1, `TestFinalize_MergeJudgement` row "a merge the conflict session resolved".
   Once `sessions` and `conflicted` are gone, this row's input (a resolved, not-up-to-date result from `eventResolver`, `internal/landingshed/parentalign_test.go:40-61`) is identical to the "non-overlapping change" row.
   Production no longer distinguishes the two, so the row repeats that one and shows neither that a conflict session ran nor that it ran under the lock.
   The behaviour is reached by `internal/landingshed_test#TestFinalize_OverRealHub` (one session, Done in one call).
   If the under-lock clause needs a row, it should set `eventResolver.lockHeld` and assert `heldAtMerge`; no Finalize test asserts it today (grep of `internal/landingshed/finalize_test.go` for `heldAtMerge`/`lockHeld:` finds nothing).

4. LOW — Card 1, `TestFinalize_MergeJudgement` drops its `merge` field.
   The card names no replacement row field that sets the resolver's `AlreadyUpToDate` for the up-to-date row, or that stops `onResolve` from moving the head.
   The row needs one, for example `upToDate bool`.

5. LOW (test economy) — Card 1, new `TestFinalize_GateAndFixer` subtest "a large merge over a failing gate runs one fixer session".
   It repeats two behaviours other rows already reach.
   The `TestFinalize_MergeJudgement` overlap row reaches "a large merge proceeds to the gate with the overlap recorded".
   The existing `TestFinalize_GateAndFixer` subtests reach "a failing gate runs the fixer and a fixed change within the caps lands".
   After D1, `Finalize.gateAndFix` reads nothing about the overlap, so the subtest's input reaches no branch those rows miss.

6. LOW (test economy) — Card 4, new top-level `TestSpecFor_RestepHold`.
   An existing per-field `specFor` test with subtests already exists, `internal/loomcli#TestSpecFor_ScratchAndFrictionDir` (`internal/loomcli/cli_test.go:283`).
   The per-recipe rows could be subtests of it, or of a renamed `TestSpecFor`, instead of a second top-level test of the same function.

7. LOW — Card 2, `restepSkips`.
   `TestRestepSkips` pins the function, but no row pins that `internal/battencli#battenCLI.wire` tells `restepSkips` as `InnerRun.HandBack`.
   envkit's nil-seam check only proves the seam is non-nil (`internal/testkit/envkit/envkit.go:74`, `"InnerRun.HandBack": true`), so a `wire` still holding the old closure passes every planned test.
   The AC "Batten's binary re-step leaves a halted escalate stop alone" rests on that one wiring line.

Tool calls:
- Read `newTestDeps`' `ReadParentTips` default, `eventResolver`, `TestGoDriver_Settle`'s `blocked` builder, `specFor` test callers and `TestFinalize_LockFirstAndFastForward`'s head: my context lacked the fixture defaults the new rows depend on.
- Read the enclosing name of the existing `specFor` test and grepped for lock-held-at-merge assertions in Finalize tests: needed to judge findings 3 and 6.
