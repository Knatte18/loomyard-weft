# Round 3 fixer report

Every finding in `round-3-review.md` was fixed in the plan files; nothing was deferred.
`lyx loom validate-plan`, run with a binary built into the scratchpad, returns `ok: true` with only informational `create-new-unit` findings, and `approved:` in `00-overview.md` is untouched (`false`).

## Changes

### F1 (BLOCKING) — Card 3 broke four existing tests

`03-prime-records-only-push.md`:
- Added `internal/webstercli/sync_integration_test.go` to the Edit list, since `TestFabricSync_CommitsAtEveryRelPathDepth` fails at Card 3's commit without a fixture fix.
- Coverage gains a bullet naming both fixture corrections and the cause (`PrimeName` runs `git worktree list` in `l.AnchorPath()`):
  `TestCommit_NestedRelPath_ClassifiesWeftFileUnderRelPath` creates `<warpPath>/wts/some-task` before it calls `Commit`, and `newPairFixtureAt` creates `<code>/<relPath>` in the code worktree.
  The bullet says no case is added, so the budget statement above it stays true.
- The ImpactSummary states that `Commit` now also fails for an anchor subdirectory missing from the code worktree, and that two hand-built fixtures change.

### F2 (LOW) — both-sides statements outside Card 3's docs line

`03-prime-records-only-push.md`:
- Added `internal/fabricengine/coalesce.go` to the Edit list.
- The Docs paragraph now names `weft_verbs.go`'s file header and the `push` and `sync` `Short` strings, `commit.go`'s `Commit` doc, and the `coalesce.go` sentence that says the detached child always supplies both paths.

### F3 (LOW) — dangling citation of the deleted test

`02-rollback-created-warp-branch.md`: the `livestate_verbs_test.go` bullet now also has the `addCase` doc comment stop citing `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites`.

### F4 (LOW) — token-carrying ownership paragraph in `doc.go`

`02-rollback-created-warp-branch.md`: the docs paragraph now says the `doc.go` paragraph on token-carrying ownership kinds also covers the created-branch proof and its reliance on the composite-literal ban in `destructiveguard_test.go`, and that its heading stops stating a count of kinds.

## Deferred

Nothing.
