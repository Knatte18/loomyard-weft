```yaml
lens: guard-reach
outcome: reported
```

Findings, guard-reach lens:

1. MEDIUM — Card 1, the retry acceptance criterion ("the retry after a merge-in-required error merges, gates and lands a code-side parent commit instead of rerouting"). The new `TestFinalize_MergeInRequiredRetry` row never reaches an overlapping parent change on the retry pass. The card says each pass reads an overlap, but that cannot happen with a fixed tip:
   - The first pass writes the merge record with `MergedTip` set to the new local tip `p2` (`internal/landingshed#Finalize.mergeParent`, the record write).
   - On the retry, `judgeParentMerge` measures the parent side from `mergedTipRead(before)`, which is now that record's `p2`.
   - `ReadParentTips` still answers `p2`, so `parentFrom == newTip` and the early return gives no overlap (`internal/landingshed#Finalize.judgeParentMerge`, the `parentFrom == newTip` condition).
   - So the row only covers the first pass's large merge, which other rows already cover.
   - The retry's own former reroute (`finalize.go:320`, "a code-side commit that arrived between the check and the parent-side merge reroutes there") stays unreached.
   - Fix: script `ReadParentTips` to answer a further tip `p3` on the read after the first parent-side merge, with `ChangedPaths` overlapping for `p2..p3`. The retry then reads a fresh overlapping parent change and must still land.

2. LOW — Card 1, the large-merge rows `TestFinalize_MergeInRequiredRetry` (new row) and `TestFinalize_GateAndFixer` (new subtest). Both tell a `ReadParentTips` with "a local tip p2 / a moved local tip" and do not say the answer keeps a valid `Relation`.
   - `Finalize.syncParent` calls `ReadParentTips(true)` before the merge whenever `PushSkipped` is false.
   - On a zero `Relation` its switch returns the "unknown relation" error (`internal/landingshed#Finalize.syncParent`, the default case).
   - The row would then stop before the merge and never reach the large-merge path it claims.
   - `newTestDeps`'s answer sets `Relation: fabricengine.ParentEqual` (`internal/landingshed/publish_test.go:71`), so a replacement must keep it. The card should say so.

Tool calls:
- grep of `"reason"` in `internal/shedverbs` non-test files: to confirm a blocked step envelope carries the halt reason, so card 3's envelope-path `JudgesFirst(producer, stop.Reason)` can match. It does: `step.go:209` lists `reason` among the short envelope keys. My context lacked the step envelope's key set.
- Read of `internal/landingshed/publish_test.go:68-80` and a grep of `parentguard.go`: for `newTestDeps`'s `ReadParentTips` answer and whether the guard reads tips. My context lacked the fixture's tip answer, which findings 1 and 2 rest on.
