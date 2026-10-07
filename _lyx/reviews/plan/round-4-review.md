---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-prime-records-only-push.md:36
    summary: Card 3 enumerates the comments that state a both-sides push but omits fabriccli/spawn.go (in no card) and the header and PushPairAnchored doc in pushanchored.go, which it edits.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/02-rollback-created-warp-branch.md:68
    summary: Card 2 edits add_rollback_adopt_test.go but does not name the TestAddRollback_AdoptedWeftBranchSurvives comment that says an empty prefix makes rollbackAdd refuse the warp-branch deletion.
---

## Verification of the round-4 focus questions

Each answer below was checked against the repository, not against earlier review text.

**1. Fixtures whose anchor subdirectory exists only on the records side.**
Not a defect beyond what Card 3 already names.
`PrimeName` runs `git worktree list` in `l.AnchorPath()` (`internal/fabricengine/worktreelist.go:147`), so a `Commit`, `PushPairAnchored` or `sync` call fails when `<worktree>/<AnchorRel>` is missing from the code worktree.
Every caller with a non-`.` anchor was traced:
- `TestCommit_NestedRelPath_ClassifiesWeftFileUnderRelPath` (`commit_integration_test.go:429`) records `wts/some-task` and builds the directory only on the records side; Card 3 names it and the fix.
- `newPairFixtureAt` (`internal/webstercli/sync_integration_test.go:79`) creates only `records/<relPath>` and its callers are `TestFabricSync_CommitsAtEveryRelPathDepth`'s rows; Card 3 names it and the fix.
- `TestCommitWeft_MachineLocalArtifactsNeverEnterWeftTreeAtAnyDepth` (`weftgit_exclude_test.go:208`) anchors at `backend` and `wts/some-task` through `hubforge.NewHub`, whose code template carries both directories (`internal/hubforge/hub.go:73-96`), so the directory exists in the code worktree and in every pair forked from it.
- `remove_junctions_integration_test.go`, `hubworkspace_test.go` and the other `AnchorRel` users never call `Commit`, `PushPairAnchored` or `sync`.
- The plain fixtures behind `NewCommitFixtureForTest` and `newPairFixtureAt(t, ".")` use anchor `.`, are their own main worktree, and resolve as the prime, so the push-recorder assertions that only count calls (`commit_partial_integration_test.go`, `commit_lock_integration_test.go`) stay green.

**2. Statements that state the old rollback or both-sides rule.**
Two gaps, F1 and F2.
Verified as named by the falsifying card: `add.go` header, `rollbackAdd` doc and step-5 logger line, `doc.go`'s rollback sentence and its token-carrying paragraph, `battencli/wire.go`, `landingshed/publish.go`, `docs/overview.md:473`, `livestate_verbs_test.go:486-492`, `destructivegaps_integration_test.go`, `coalesce.go:80`, `commit.go:74`, and `weft_verbs.go`'s header and `Short` strings.
`doc.go:530` ("origin is never touched") belongs to `Reconcile`, not `Add`, and `PATTERN-batten-bookend.md` spells only `RequireWarpWorktree` (not renamed) and prose, so neither needs a change.

**3. Surviving citations of deleted tests.**
Not a defect.
The only test function a card deletes is `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites`; its only citation outside its own doc comment is the `addCase` doc in `livestate_verbs_test.go:412`, which Card 2 names.
The remaining occurrence is in `docs/benchmarks/`, a measurement report that is never rewritten.
The dropped `TestCheckFork` case and the dropped `vocabscan_test.go` row are cited by no comment or doc.

Beyond the focus, three mechanical cross-checks held: every file that spells an old hubforge accessor, fabricengine export, `WEFT_SKIP_*`, result key or `side=` detail is in some card's list (`PairWarpWorktree` through `ResetPairWarp`, `warp_worktree`, `warp_branch_*`, `side=warp`, `side=weft`), every reader of the `Fabric` struct's `warp` and `weft` fields is in Card 8's list, and none of the new names already exists in the module.
`## verify:` follows the template's stated order.

### [MEDIUM] Card 3 omits falsified both-sides statements in `fabriccli/spawn.go` and `pushanchored.go`

**Location:** `_lyx/plan/03-prime-records-only-push.md:36-40`; `internal/fabriccli/spawn.go:1-3,10`; `internal/fabricengine/pushanchored.go:1-2,124`.
**Issue:** Card 3 lists the comments and strings that state a both-sides push and names three: `weft_verbs.go`, the `commit.go` doc and the `coalesce.go` sentence.
`internal/fabriccli/spawn.go` is in no card, and its header ("spawnPush delegates to fabricengine.SpawnDetachedPush with both worktree paths, so the detached child pushes the warp side and the weft side") and `spawnPush` doc become false once the `sync` verb passes an empty code-side path in the prime.
`pushanchored.go` is in Card 3's Edit list, but its header ("push the warp worktree with it") and the `PushPairAnchored` doc ("pushes the unpushed commits of l's code worktree and of its paired records sibling ... The code side is pushed first") are not named, and the doc is the first statement a reader of the new `code_push_skipped` field consults.
`doc.go`'s `PushPairAnchored` sentence ("the second pushes the code side and then the records side", near line 583) is covered only by Card 3's generic "push and spawn text".
**Fix:** In Card 3, add `internal/fabriccli/spawn.go` to the Edit list.
Extend the list of comments that change with: the `spawn.go` header and `spawnPush` doc (the child pushes whichever side is supplied, the code side being empty in the prime), the `pushanchored.go` header and `PushPairAnchored` doc (prime: records side only, the skip reported as `code_push_skipped`), and `doc.go`'s `PushPairAnchored` sentence.

### [LOW] Card 2 leaves the empty-prefix refusal comment in `TestAddRollback_AdoptedWeftBranchSurvives` unnamed

**Location:** `_lyx/plan/02-rollback-created-warp-branch.md:68-71`; `internal/fabricengine/add_rollback_adopt_test.go:82-87`.
**Issue:** The comment above `const branchPrefix = "task/"` says `ownedManagedBranch` needs a `-weft` suffix or a configured prefix, "so rollbackAdd's warp branch deletion would be refused as unmanaged with an empty prefix", and that the non-empty prefix is there to make the branch recognisable to the gate.
D2 makes the rollback prove ownership by creation in this call, so the deletion no longer depends on the prefix and the comment states the old rule.
Card 2 edits this file and names three other tests in it, but not this one.
**Fix:** In Card 2's `add_rollback_adopt_test.go` bullets, add that `TestAddRollback_AdoptedWeftBranchSurvives` drops the comment (and the prefix it motivates, or restates the prefix as incidental), so the file carries no statement that an empty prefix blocks the warp-branch deletion.
