```yaml
lens: code-scope
outcome: reported
```

**Lens: code-scope. Per-change trace, then per-decision coverage.**

The plan cards were not in my context and I did not re-read them. I traced through the decision record's D-headings and its Takes list.

**Changes → decision**

| Change (files) | Decision |
|---|---|
| `lymcwd` `OneRepository`, `readRecordedAnchor`, `ResolveWithin`; fabric `onerepository.go` (`oneRepositoryAt`, `recordsBranchIn`, `pairSlugOfRecords`, `recordsLockDir`) | D1, D3 |
| `clone.go`, `clone_onerepository.go`, `boardweft.go`, `fabriccli` clone/`--single-repo`/docs, `PushOneRepositoryClone`, `PrimeBranchConfigKey` | D2 (the config key is D7) |
| `ProtectedBranchRefusal` moved from `landingshed` to `pushclassified.go`; `gitrepo` `remoteLines` | D2 (forced: the protected match must see hook text) |
| `add.go`, `forkrecords.go`, `junction.go` (`wireOneRepositoryDirs`), `drift.go`, `reconcile.go`, `unwire.go`, `weftwiring.go`, `worktreelist.go`, `status.go`, `anchorchange.go` unwire, `rollbackAdd` | D3 (anchor-change parts also D5) |
| Every `RecordsBranchName` → `recordsBranchIn` caller swap (checkout, cleanup, configchanges, destroy, originreads, pairbranch, remoteleftover, …) | D3/D6 (forced: no second branch) |
| `recordscommit.go`, `recordsgit.go`, `commitweftpaths.go`, `commit.go`, `recordsduringmerge.go`, `MergeStateOperation`, `refuseOffPrimeBranch` | D4, D7 (prime-branch refusal) |
| `statuscommit`, `battencli`/`loomcli` wiring, `runfsm` `PushOwedWayForward`, `pairteardown` `runread` | D4 (wait), D7 (owed and protected classes) |
| `merge.go`, `mergerecords.go`, `mergelifecycle.go`, `mergeerrors.go`, `mergeguards.go`, `mergestate.go`, `midmerge.go`, `restoreRecordsToHead`, `MergeStartNoFastForward`, `FastForwardPair` leveling | D5 |
| `loomcli/start.go`, `landingshed` `premerge.go` and `finalize.go` for `MidMergeRecordsUnsettled` | D5 (forced by the new kind) |
| `recordspaths.go` (`RecordsPathsAt`, `IsRecordsOnlyCommit`, `IsFabricMergeCommit`, trailer constants) | D5, D7 |
| `pushanchored.go`, `coalesce.go`, `spawn.go`, `pushclassified.go`, `failedpush.go`, `handover.go`, `codetipsettle.go`, `takeoverwrite.go` | D6 |
| `remoteteardown.go`, `remove.go`, `destroy.go` `deleteViewRefOnOrigin`, `ViewRefPrefix` | D6 |
| `replay.go`, `integrateweft.go`, `pull.go`, `originreconcilemoves.go`, `cherryPickReworded`, `primepushguard.go`, `PushFailureOperatorUnpushed`, `PushFailureProtected`, `IsPushProtected` | D7 |
| `boardweft.go` remote-tracking adopt, `seedDotLymExclude`, the Bolt exclude in `commitWeftAt`, `boardengine` test | D11 |
| `hubforge` `NewOneRepoHub`, `SeedConfig`, `ReanchorPair` guard, the new one-repo integration tests | D12 |
| `cmd/lym` allowlists, `gitrepoboundary` rename, fabric `doc.go`, `gitrepo/doc.go`, `docs/overview.md`, `README.md`, `docs/getting-started.md`, `PATTERN.md`, `pattern/*` | Forced or declared (D2, Constraints) |

**Decision → builders**
- D1–D7, D11, D12 each have builders, per the table.
- F16 (build into the current fabricengine) is honoured.
- The operator's "motif writes design only" entry governs the earlier motif, not this build.
- D3's "hub workspace file lists no sibling folders" has no change to `hubworkspace.go`, and no test asserts it (see finding 3).

**Findings**

1. **MEDIUM, class `scope`**: the Fabric-Merge trailer is applied to every fabric merge in both layouts.
   - Location: `internal/fabricengine/mergelifecycle.go` (`concludeMergeSides` calls `mergeCommitMessage` unconditionally).
   - D5 and its Delivers list scope the merge trailer to a one-repository hub, and D12 says the two-repository tests stay as they are.
   - The diff changes two-repository behaviour, and it edits an existing two-repository assertion: `landingshed/finalize_integration_test.go` `wantMsg` now ends in `Fabric-Merge: task`.
   - No `Pattern-Departure:` trailer appears in the range log, and `doc.go` states "in either layout" as the intent.
   - Evidence: the `mergelifecycle.go` hunk (lines 48–55 of the diff) and the `finalize_integration_test.go` hunk.

2. **LOW, class `scope`**: the anchor-change trailer is added in the two-repository move as well.
   - Location: `internal/fabricengine/anchormove.go`, `moveWeftContent` (`appendTrailerLine(..., AnchorChangeTrailerKey, to)`).
   - D5's anchor-change rule concerns a one-repository records commit; `doc.go` says "Every layout's anchor-change commit carries the trailer".
   - I read it as an edit forced only by the marker-change guard, which runs in one-repository merges alone, so the two-repository trailer is not forced.

3. **LOW, class `scope`**: D3's "the hub workspace file lists no sibling folders" is built by no change and pinned by no test.
   - The git diff shows no change to `hubworkspace.go`, `scratchlink.go` or `gitexclude.go`, and `hubworkspace.go` has no `oneRepositoryAt` branch.
   - `hubWorkspaceFolders` at `hubworkspace.go:317` may already list nothing extra because `RecordsWorktree` returns the worktree itself.
   - I did not read it, so this is unverified.
   - `scratchlink.go` and `gitexclude.go` are Takes-listed with no diff, but the `.scratch` link is covered by the `ScratchLinksToTheWorktreesOwnDotLym` row and the exclude by `seedGitExclude`.

4. **LOW, class `scope`**: `MergeStateOperation`'s two-repository branch now also reports cherry-pick, revert and rebase.
   - Location: `internal/fabricengine/recordsduringmerge.go:157-184`, which calls `gitOperationAt` before the merge-head check.
   - D4 scopes that classification to the one-repository seam, and `MergeStateActive` is now `operation != ""` for both layouts.
   - A two-repository status seam therefore starts skipping on those operations, which it did not before.
   - Only the "any merge state of the records worktree blocks" part of the two-repository behaviour is documented as kept.

**Tool calls**
- 1. `git diff --stat` over `hubworkspace.go`, `scratchlink.go`, `gitexclude.go`, `bolt.go` and `trailercommit.go`, plus a grep of `hubworkspace.go`: my context held no diff for those Takes-listed files.
