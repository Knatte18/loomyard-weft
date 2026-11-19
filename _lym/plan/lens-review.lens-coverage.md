```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the plan in `_lym/plan/` (cards 1–10): every decision and acceptance criterion traced to a card and a planned test row.

**BLOCKING**

1. **Card 5, the one-URL check — an existing test breaks.**
   - Every one-URL clone now runs `readOriginLayout`'s `ls-remote` before `probeWeftBinding`. An unreachable URL therefore fails there first.
   - `TestCloneHub_UnreachableWeftIsHardError` requires the `"probe weft "` prefix in both argument forms (`internal/fabricengine/warpbinding_clone_integration_test.go:111` and `:142`). The comment at `internal/fabricengine/clone.go:227` says the same.
   - Card 5 does not list that test and does not give `readOriginLayout`'s error wording. Either the wrap keeps the prefix or the test joins card 5. The unreachable-origin branch of the new check has no planned row of its own either.

**MEDIUM**

2. **Card 2, `originNamespaceAt`: the "no `origin` remote answers identity and is cached" branch has no row in `TestOriginNamespaceAt`.** Its row list covers missing path, no `.git`, default refspecs, namespaced, the cached second read, created-after-missing and an empty `.git`.
3. **Card 5, the join path: two branches have no row.**
   - A `.lym-warp` naming another URL, or a wrong `--shortname`, should refuse and tear the hub down through `resolveEffectiveWarpURL` / `resolveEffectiveShortname`. Card 7's `TestCloneOneURL_JoinsTheNamespacedHub` covers only the success path.
   - `--reset` with an explicit `--subpath` on a join runs `checkSubpathBeforeReset` with `oneRepositoryNamespace()`. No row reaches it. That leaves the namespaced `recordedAnchorOnWeft` and `fetchWeftBranchTip` branches untested.
4. **Card 5, the seed path: "the shortname is required, as a fresh bind's are" has no row.** No `--single-repo` clone without `--shortname` is planned.
5. **Card 5, `PushCloneBoard`: three teardown branches are unreached.**
   - A lost lease on the records push ("any other cause, a lost records lease included: run the same clone again") has no row.
   - In the refused row of card 7's `TestCloneSingleRepo_FailedSeedPushLeavesNothing`, the hook refuses every `refs/lym/` update. The records push therefore never lands, and the refusal-after-records-landed restore is never reached. Only the lost board lease reaches the restore.
   - A failed restore ("named in the error beside the cause") has no row.
6. **Criterion D3, "a rerun with the same or another `--subpath` replaces the leftover records branch".**
   - The same-`--subpath` rerun runs only after the refused row, over an origin with no leftover (the hook let nothing land).
   - Only the another-`--subpath` rerun meets a real leftover (the "leftover" row).
   - A same-subpath rerun over a leftover `refs/lym/heads/main-weft` is not planned.
7. **Shared Decision coverage-of-namespace-sites / D4 "add, push, pull, replay, hand-over…" / D8 "the two-repository suites' key flows".** Card 9 tests add, push, remove and archive, reconcile, cleanup and merge only. These one-repository paths that cards 3 and 4 change have no row:
   - pull: `weftAnchorStopBeforePull`, `settleUnmovedWeft`.
   - replay after a rejected push: `retryWeftPush`, and `retryBoltPush`'s namespaced `PushRebaseFree`.
   - hand-over and takeover: `takeoverBranch.land`, `Fabric.PushWeft`'s `PushCoalesced`.
   - adopting a pair's records branch from origin: `resolveWeftBranch`, `Topology.Add`'s namespaced records fetch, `probeWeftLeftover` and `probeLiveLocalWeft`.
   - other reads: `targetWeftTip`, `RecordsBranchFile`, `readSideOwnerArchives`.

   The decision says card 9 reaches these paths; its rows do not.
8. **Card 1, `remoteTrackingRef`: the "remote with no configured fetch refspecs keeps today's `refs/remotes/<remote>/<short>` answer" fallback has no row** in `TestUpstreamSHA` or `TestParity`.
9. **Card 8, `primaryWeftBranch`: "a failed layout or config read is refused the same way, naming the error" has no row.** Only the unset key is planned.

**LOW**

10. **Card 3, `DeleteRemoteBranch`: the probe's exact-line match has no row.** A namespaced `ls-remote` pattern could tail-match a longer ref such as `refs/lym/heads/x-weft` against `.../y/x-weft`. That miss would go unseen.
11. **Card 9, `TestOneRepositoryPair_RecordsLiveInTheNamespace`: the "adds no ref under `refs/tags/`" assertion holds only if card 3's `pushArchiveTag` uses an explicit `refs/tags/<t>:<ns.Tag>` refspec.** The row is fine. The bare-push hazard D2 forbids ("never a bare `git push` in the records store") has no guard row of its own.

Tool calls:
- One grep for remote-HEAD reads (`remoteDefaultBranch`, `warpDefaultBranch`, `--symref`), the card 8 test names, and `"probe weft"` assertions. I needed it to confirm card 4's no-records-HEAD-read claim (it holds), check that card 8's tests exist, and check whether the unreachable-clone test pins the prefix.
