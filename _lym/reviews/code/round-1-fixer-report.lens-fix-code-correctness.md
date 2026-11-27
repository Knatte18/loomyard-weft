```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the fix's changed code, branch by branch. I traced the inputs through the code as shown in context and found no wrong result and no broken caller.

1. **LOW**: `TestDarnDeps_BranchWorkSeams` (`internal/loomcli/subjects_integration_test.go`) cannot tell `taskMergeBase` from `pairMergeBase`.
   - **What passes with the defect:** the new pair has no merge record and no parent merge, so both readers return the fork point. Wiring `BranchCommits` to `pairMergeBase(l)` would still pass.
   - **Why it matters:** with a merged parent tip that the local parent branch has not reached, that wiring lists the parent's commits as the task's own. F3's summary names "CommitLog from taskMergeBase to HEAD" as an unpinned part.
   - **Evidence:** `taskMergeBase` calls `fallback()`, which is `pairMergeBase`, when no record is found (`internal/loomcli/subjects.go:136-137`). `mergeUnreachedParentTip` and `writeMergeRecordNaming` in the same test file already build the case that separates them.

2. **LOW**: `renderPriorWork` (`internal/loomshed/darnwrite.go`) now lists directories under text that calls them files.
   - **Concrete input:** a `.scratch` holding only an empty `runs/` renders "may have left the files below" and then "These files survive under the scratch directory …:" followed by `- runs/`. No file survives in that case.
   - **Why it is new:** the old walk skipped directories, so this input rendered `(none)`.
   - **Evidence:** `listScratchEntries` appends every direct child, directories included. The two header strings still say "files".

Traces with no finding:
- **`listScratchEntries`:** an absent directory or a dangling link returns nothing. A regular file at the path returns an error. An unreadable subdirectory is listed by name and not read. A `ReadDir` error discards the partial result.
- **`renderPriorWork`:** with an outcome and nothing listed, no later-spawn line appears. The commits-only, scratch-only and empty cases each reach the intended branch.
- **Tests:** the changed `TestRenderPriorWork` rows, the `runs/` row and the unreadable-subdirectory row would fail on the old code. The dangling-link and unlistable-path rows add coverage and pin no defect.

Further tool calls: none.
