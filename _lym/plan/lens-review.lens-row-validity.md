```yaml
lens: row-validity
outcome: reported
```

Row-validity lens review of the 10-card one-repository namespace plan.

**Findings**

1. **BLOCKING** — Card 5 (`refuseOneURLClone` order, D3). The refusal order sends a real records repository to the wrong way forward.
   - The card checks f06acf39's layout first ("with `--single-repo` a default branch that tracks a root `.lym-anchor` or `_lym`") and only then the `.lym-warp` records-repository refusal.
   - A two-repository weft whose board tracks the legacy root `.lym-anchor` holds both `.lym-warp` and `.lym-anchor` on its default branch. Card 5's own `TestCloneHub_AnchorAdoptPath` row names exactly that weft.
   - A `--single-repo` clone of it therefore gets the f06acf39 way forward: "remove `.lym-anchor` and `_lym` from the code branches". That tells the operator to strip the records repo's anchor, when the right answer is a two-URL clone.
   - The planned row "one whose default branch holds `.lym-warp`" builds a bare with `.lym-warp` alone, so it passes and never reaches this case.
   - Fix: check `.lym-warp` before the root-anchor/`_lym` check, and add a row whose default branch holds both `.lym-warp` and `.lym-anchor`, asserting the two-URL way forward.

2. **MEDIUM** — Card 9 `TestOneRepositoryOrigin_ReconcileAndCleanupSeeTheNamespace` (D2/D4 remote cleanup). The row drives the wrong entry point.
   - `Topology.Cleanup` (`internal/fabricengine/cleanup.go:130`) never lists origin's branches or tags.
   - The records-store remote listings are in `Topology.CleanupRemoteWarp`: `remoteBranchTips(weftRoot)` at `cleanupremotewarp.go:96` and `remoteTagRefs(weftRoot, …)` at `:100`. `pairbranch.go:211` also lists the records store.
   - The row also names no observable for "lists no code branch". The failure it should catch is the sweep treating origin's code `refs/heads/main` as a records branch.
   - Fix: drive `Topology.CleanupRemoteWarp` and assert on its result entries, plus origin's `refs/heads/main` surviving an apply.

3. **MEDIUM** — Card 7 `TestCloneSingleRepo_FailedSeedPushLeavesNothing` (D3 rerun criterion). The refused row never makes a leftover, so its rerun proves nothing about replacement.
   - The hook refuses every `refs/lym/` update, so the records push fails before landing. Origin keeps no leftover, and the reruns run over a clean origin.
   - The only leftover row reruns "under another `--subpath`". The criterion "with the same or another `--subpath`, replaces the leftover records branch" has no same-subpath leftover row.
   - The refused row also never exercises "records landed, then the board push refused, then restore". Only the lease-lost rows reach the restore.
   - Fix: add a same-subpath leftover row, and refuse only `refs/lym/heads/main` in the hook so the restore after a refused board push is reached.

4. **MEDIUM** — Card 7, the same test. Its input has no listed helper.
   - It "drives `fabriccli.CloneAndWire` with `SingleRepo` over a bare copied from the template". `copyBares` and `buildBareTemplate` are unexported in hubforge (`hub.go:93`, `:31`), and `fabricengine_test` cannot reach them.
   - The card declares no exported copy helper, and `NewOneRepoHub` always clones, so it cannot hand back an un-cloned bare or one carrying a hook.
   - Fix: declare a hubforge helper that returns a fresh copy of the code bare, or say the test builds its bare with `gitkit`.

5. **MEDIUM** — Card 2 `TestOriginNamespaceAt`. Two stated branches have no row.
   - "A failed config read is not cached": the empty-`.git` row asserts only the error. It does not repair the store and read it again, as the removed `FailedReadNotKept` row did (`13cebe3d^:internal/fabricengine/onerepository_cache_test.go`).
   - "A repository with no origin remote answers identity and is cached": no row.

6. **LOW** — Card 1 `TestUpstreamSHA`. The "merge ref outside every refspec answers `ErrNoUpstream`" row does not fail if the code is wrong.
   - The current `UpstreamSHA` (`internal/gitrepo/blobread.go:305ff`) already answers `ErrNoUpstream` for any merge ref outside `refs/heads/`, because `!branch.Merge.IsBranch()`.
   - The row only fails on old code if the merge ref is a `refs/heads/<b>` that no refspec of a lym-only store maps, with a `refs/remotes/origin/<b>` present; the old code returns that ref's SHA.
   - The branch "a remote with no refspecs keeps today's answer" has no row either.

7. **LOW** — Card 7 `TestCloneSingleRepo_SeedsTheNamespace`. "Every `refs/heads/` ref of the bare is at its tip before the clone" checks the precondition only. The criterion "leaves every `refs/heads/` ref at its prior tip" needs a before/after comparison.

8. **LOW** (test economy) — Card 9 `TestRemove_NeverPushedRecordsBranchRecordsNoRemoteDeletion`, two-repository row. It repeats `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote` (`internal/fabricengine/cleanupremote_integration_test.go:195`).
   - Both reach the `deleteRemoteBranch` executor with a never-pushed records branch and assert no `KindRemoteBranchDeleted` entry.
   - Fix: make the one-repository case a row of that existing test instead of a new top-level test.

9. **LOW** (test economy) — The `--single-repo` with two URLs refusal is planned twice.
   - Card 5 has a "second URL" row in `TestCloneHub_OneURLRefusals`, and card 6 adds a row to `TestCommand_RefusesBeforeAnyHubWork`. Both reach the same offline check in `CloneHub`.
   - Keep one: the CLI row if the envelope matters, otherwise the engine row.

10. **LOW** — Card 4 lists the glyph `internal/fabricengine#Topology.cleanupRemoteWarp`, which does not exist. The method is the exported `Topology.CleanupRemoteWarp` (`internal/fabricengine/cleanupremotewarp.go:65`).

11. **LOW** — Card 8 `TestCleanup_ProtectsPrimaryWeftBranchAfterCheckout` is not a table today (`internal/fabricengine/cleanup_primary_integration_test.go:25`).
    - "Gains a one-repository row" means converting it to a table over two hub builders.
    - The existing body moves the prime with raw `git checkout -b`, not `lym fabric checkout`. The criterion's "after `lym fabric checkout`" is therefore met only by that stand-in, and the card should say so.

The other rows reach their branches and would fail on wrong code:
- card 1: `TestRefNamespace`, the namespaced `TestUpstreamSHA` row and the `TestParity` Upstream scenario (an "Upstream agrees" row exists at `parity_test.go:299`);
- card 3: the `TestDeleteRemoteBranch` namespaced and leased rows, the `TestUpdateRemoteBranchLeased` empty-expect rows (the existing invalid-SHA row at `push_test.go:407` holds no empty value, so it stays valid), the `TestPushRebaseFree` namespaced row and the `TestRetryablePushRefusal` row;
- card 7: the lease-lost restore rows;
- card 8: the `master` row, where the old code protects `main-weft`, not `master-weft`, and fails.

Tool calls made (five of six):
- grep of test function locations and packages, plus `KindRemoteBranchDeleted` uses: my context lacked the existing tests' files and packages.
- `cleanupremote_integration_test.go:185-232`, `cleanup_primary_integration_test.go:1-120` and `deleteremotebranch_integration_test.go:1-140`: to read the rows the cards extend and find repeats.
- `push_test.go` lease rows, `parity_test.go` scenarios, `TestRemoteURL`, and the `Cleanup` and `ReconcileOrigin` signatures: to check whether the new rows conflict with existing ones or fit them.
- grep of the `remoteBranchTips`, `remoteTagRefs` and `remoteDefaultBranch` callers and the parity Upstream rows: to find which entry point lists the records store remotely.
- (No sixth call made.)
