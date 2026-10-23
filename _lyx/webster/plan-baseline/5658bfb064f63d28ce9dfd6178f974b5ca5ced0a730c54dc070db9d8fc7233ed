# Card 1 — fabric-merge-in-at

**Create:**
- `internal/fabricengine#Fabric.MergeInAt`
- `internal/fabricengine#Fabric.mergeIn`
- `internal/fabricengine_test#TestMergeInAt_MergesTheGivenTipNotTheLocalOne`

**Edit:**
- `internal/fabricengine#Fabric.MergeIn`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#resolveMergeSources`
- `internal/fabricengine_test#newMergePairFixture`

**Intent:** Give fabric a merge-in of a given parent SHA (D2).
Move `Fabric.MergeIn`'s body into the new unexported `Fabric.mergeIn(source, warpSHA)`, which `MergeIn` calls with an empty SHA and the new exported `Fabric.MergeInAt` calls with the SHA it is told.
With an empty SHA, `mergeIn` behaves exactly as `MergeIn` does today: `resolveMergeSources` resolves the code side from `source` with its freshness pick.
With a SHA, `mergeIn` still calls `resolveMergeSources` for the records-side SHA and its guard reasons, then replaces the code-side SHA by the told one, after checking that the told SHA resolves to a commit in the pair's code repository; a SHA that does not resolve is the same `mergeReasonSourceNotFound` guard reason the name path raises.
The merge state's `Source` still names `source`, and `WarpSource` records the told SHA, so a later `MergeContinue`, `MergeAbort` and the pre-merge step's parked-merge probe read the merge as a merge-in of `source`.
`MergeIn(source)` stays as it is for every other caller.

The new integration test builds the merge pair fixture and runs three rows.
In the first, a local `feature` branch is one commit ahead of `origin/feature`; `MergeInAt("feature", <origin/feature's SHA>)` brings in the upstream commit's file and not the local-only commit's file.
In the second, the told tip changes a file the pair's HEAD also changed, so the merge conflicts and parks; the parked state read through `MidMerge` names `feature` as its source and the told SHA as its code-side source.
In the third, the told SHA names no commit in the pair, and the call returns the merge guard error carrying `mergeReasonSourceNotFound`, with nothing merged.

`doc.go`'s merge-surface section gains one paragraph on `MergeInAt`.

**ImpactSummary:** One new exported verb and a private body shared with `MergeIn`; `MergeIn`'s behaviour and every existing caller stay unchanged.
