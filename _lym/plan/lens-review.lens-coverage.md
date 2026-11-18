```yaml
lens: coverage
outcome: reported
```

Coverage lens over the plan's decisions, criteria and the branches the cards introduce. I found two defects that break existing behaviour and one that leaves the prime unguarded, plus a set of untested branches.

**Blocking**

1. **BLOCKING — Cards 1 and 3 (D1): the layout predicate also fires in two-repository hubs.**
   - In a two-repository hub, `.lym-anchor` sits at the weft sibling's root (`internal/lymcwd/anchor.go:158-161`, "at the root of its records sibling"). So `OneRepository(siblingRoot)` answers true there.
   - Card 1's `ResolveWithin` takes the one-repository branch whenever `OneRepository(workTreeRoot)` holds. A cwd inside the sibling then stops resolving to its owner, which breaks `internal/lymcwd_test#TestResolveWithin` rows `SiblingRecordsDirectory` and `SiblingRoot` (`internal/lymcwd/lymcwd_test.go:350`, `384-385`). Card 1's new `readRecordedAnchor` first read also reads a sibling's own root marker.
   - Card 3's `RecordsLockDirPath(path)` and `ensureWeftLockDirAt` switch on "a path whose root carries the marker". They are always called with the records worktree, which is the weft sibling in a two-repository hub. Every two-repository write lock, push lock and exclude seeding would move to `<gitdir>/lym-locks`.
   - Card 3's `TwoRepository` row ("no root marker") does not model a real sibling, so it would not catch this.
   - The predicate must be read on the code worktree root, or the cards must exclude the sibling.
2. **BLOCKING — Card 10 (D4): `MergeStateOperation` drops today's conflicted-index probe.**
   - `MergeStateActive` today is `MergeHeadPresent() || len(ConflictedFiles()) > 0` (`internal/fabricengine/mergestateactive.go:42`).
   - Card 10 makes it `operation != ""`, where the operation is read only from `MERGE_HEAD`, `CHERRY_PICK_HEAD`, `REVERT_HEAD` and the rebase directories.
   - A conflicted squash has no `MERGE_HEAD`, so it now reads false. That breaks the existing `internal/fabricengine_test#TestMergeStateActive` row "weft conflicted squash without MERGE_HEAD reports true", which card 10 lists under Edit without addressing it.
3. **BLOCKING — Card 14 (D7): the prime guard covers only `PushPairAnchored`.**
   - `Fabric.Commit` and `lym fabric sync` in the prime spawn the detached push (`SpawnDetachedPush` → `DetachedPush` → `coalescePushSides` → `pushSideDetached` → `pushClassified`). None of these is edited by card 14.
   - So a records push of the prime's branch can still carry operator commits. D7 says "A records push of the prime's branch … pushes only when every commit origin lacks is …".
   - No row drives the detached push.

**Wrong behaviour and fidelity**

4. **MEDIUM — Card 5 (D3): the Bolt commit writes the forbidden exclude lines into the one repository.**
   - In a one-repository hub, `_bolt` is a worktree of the one repository.
   - `commitWeftAt` calls `seedWeftArtifactExcludes` directly (`internal/fabricengine/recordsgit.go:373`), and so does `CloneHub` after `ensureBoardWorktree`. That writes `.weft/`, `.lym/` and the legacy lock names into the shared exclude, which D3 forbids.
   - No card edits these sites. Card 5's own row `SingleRepoOnAFreshOrigin` asserts "no `.weft/` exclude line", so it would fail.
   - Related: `lym-bolt` itself carries `.lym-anchor` (D2), so `OneRepository(_bolt)` is true. No card says how fabric treats that.
5. **MEDIUM — Shared Decision `layout-read` vs D1: fabric reads the marker from disk, not from git.** D1 says "`lymcwd` reads the marker file from the worktree's root, and fabric reads it from the branch in git". The plan has fabric call `lymcwd.OneRepository` on disk, and adds an unlicensed rule: a slug with no worktree yet takes its hub's layout.
6. **MEDIUM — Card 5 (D2): only the marker is committed before the push.** D2 says `--single-repo` "commits the anchor marker and seed records on the prime's branch first, then creates `lym-bolt`". Card 5 commits only `.lym-anchor` before the push. The seed records arrive later through `CloneAndWire`'s config commit, after `lym-bolt` exists, and that commit is not pushed by the clone.
7. **MEDIUM — Card 9 (D5): `syncSideBeforeMerge` is not carried.**
   - `Fabric.Merge` brings the receiving branch level with its origin through `syncSideBeforeMerge` (fetch, then `MergeFFOnly`, refusing as diverged).
   - D5 says bringing a branch level with its own origin tip commits records, then replays local records-only commits, and refuses only on an unpushed non-records commit.
   - Only `FastForwardPair` (card 13) is changed. A parent carrying local records-only commits behind origin will refuse the landing as "branch not synced to upstream".
8. **MEDIUM — Card 9, D5 marker-change refusal: `lym fabric add --anchor` in one repository is not carried.**
   - `refuseSourceMarkerChange` admits a marker change only by `AnchorChangeTrailerKey`.
   - Add's `--anchor` path (`moveForkedWeftToAnchor` / `stageNewMarker`) writes the marker in Add's records commit without that trailer.
   - Card 6 does not handle `--anchor`, so merging such a task back refuses. No row covers it.
9. **MEDIUM — Card 7 (D7): merge step 1 bypasses the prime-branch refusal.** Card 9's step 1 calls `commitRecordsPathsHeld` directly. Card 7 hooks `refuseOffPrimeBranch` only into `CommitRecordsPaths`, `CommitEmptyWithTrailers` and `commitWeftLocked`. So a merge in a prime that is off its recorded branch commits records on the wrong branch, which D7 says must refuse.

**Unreached or untested branches**

10. **MEDIUM — Card 10, `TestRecordsCommit_DuringMerge` row `CrashBeforeStepThreeCommitsAbortsAndPushes`.** The row stops the merge with "the card 9 hook returning a crash". Card 9 declares the hook as `func()`, which returns nothing, so the row cannot stop a merge before step 3 as written. This leaves D4's crash criterion unreached.
11. **MEDIUM — Card 10: three new branches have no row.**
    - The `Fabric.Commit` branches: a records-only list passes around foreign merge state, and a list holding code paths still refuses.
    - The `commitRecordsAroundMerge` refusal for any other merge state (cherry-pick, revert, rebase, or a foreign merge with a records path unlike HEAD). Only the `MergeStateOperation` probe has rows.
    - `MidMerge` returning `MidMergeRecordsUnsettled` for a real merge. Finalize's row uses a fake `midMerge`.
12. **MEDIUM — Card 11, D4 criterion "a transition during a cherry-pick … neither skips … nor lets the run act past it".**
    - The plan relies on tier0 fakes plus card 10's probe rows.
    - It drops the real-seam rows from `internal/loomcli#TestCommitStatusSeam_Real`, arguing the sleep cannot be ended. But `Deps.Sleep` is injectable.
    - So no row drives the wired seam into a real blocking operation.
13. **MEDIUM — Card 13 (D7): several replay branches have no row.**
    - Ended-run set-aside among records-only commits.
    - The push-path stop: a rejected push with a local code commit, recorded deferred with `Commits`.
    - A takeover read ahead of that stop.
    - `reconcileRecordsBranch` reporting the branch diverged.
    - All rows are pull rows.
14. **MEDIUM — Card 6 (D3): rollback, the adopt paths and the skipped weft fast-forward and remote weft-branch replacement have no row.** They are stated as one-repository branches of `Topology.Add` and `Topology.rollbackAdd`.
15. **MEDIUM — Card 6 / D3: no row asserts the `.scratch` link target in one repository.** D3 says `.scratch` links to the worktree's own `.lym/scratch`.
16. **MEDIUM — Card 3: about thirty members switch to `recordsBranchIn`, and most have no one-repository row.** Uncovered: `Topology.Checkout`, `Topology.switchOrForkWeft`, `Topology.Status`, `ReadConfigChanges`, `Topology.RecordsBranchLive`, `Topology.inspectPair`, `Topology.keptReason`, `Topology.originOtherPrefixBranch` and `Topology.reconcileMissingWeft`.
17. **MEDIUM — Card 12, D6 criterion "teardown pushes the archive tag before deleting the branch on origin".** `TestRemove_OneRepository` asserts only the end state (tag present, branch and view ref gone). Neither the order nor the view-ref delete's place in it is pinned.
18. **MEDIUM — Card 8: the integrate-kind refusal in one repository (naming the layout) has no row.**

**Low**

19. **LOW — Card 5, `TestCloneHub_OneRepository`.** The test is named for `CloneHub`, but `CloneHub` alone does not wire junctions (`internal/fabriccli#CloneAndWire` does). The row's assertions on real `_lym` and `.lym` directories and the exclude need `CloneAndWire`, and the card does not say which entry the test drives.
20. **LOW — Card 4, `TestPushFailureClasses` row `AProtectedBranchRefusal…`.**
    - A pre-receive decline prints "remote rejected", which gitrepo's push retry treats as retryable, so the row may wait out production backoff delays.
    - Whether git's stderr survives into `err.Error()` for `ProtectedBranchRefusal` to match is not established.
21. **LOW — Card 12: `pushOwedCodeTip`'s one-repository no-op (no `deleteGivenUpCodeBranch`) has no row.** Only `ReturnToPreviousTransition` is pinned.
22. **LOW — Card 14: the act rule's refusal for an `operator_unpushed` owed push.** It still names `lym fabric push` (PATTERN-run-fsm "An owed push holds the run"), while the record's way forward is `git push` in the prime. No card touches `runfsm` or `pairteardown`.
23. **LOW — Card 5, D11 row `BoardWriteStaysOnLymBolt`.** It uses `Bolt.Commit` and `Push` directly rather than a board write through `boardengine`.
24. **LOW — Card 2: `IsRecordsOnlyCommit` on a root commit has no row** (`RootCommit` exists only in `TestRecordsPathsAt`).

Further tool calls: none.
