```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran F16's search for one-repository namespace sites that no row reaches, and checked whether the fix covers every member of that set.

Findings I left unchecked:
- F16, two members only partly checked: `originBranchHead` on the records side, reached through `pushJob.awaitOrigin` → `observeOrigin` → `branchGoneOnOrigin`, and `lsRemoteHeads` on the records side, reached through `originreconcile.go:507`. I did not confirm that card 7's push and reconcile rows go through these paths in a one-repository hub. A namespace-blind lookup in `awaitOrigin` would probably read the records branch as gone, which would affect the push rows, but I did not confirm this.

Findings:

- **MEDIUM**, on `00-overview.md`'s coverage-of-namespace-sites list and card 7's new bullets. The set misses `resolveArchiveTip`, a fifth card 3 site that no one-repository row reaches.
  Evidence:
  - Card 3 changes `resolveArchiveTip` to run `ls-remote origin &lt;ns.Branch(b)&gt;` (`03-namespaced-remote-reads.md`, the Edit list and its first Intent bullet).
  - Its only caller is `archiveWeftTip`, which calls it only when `tip == ""` (`internal/fabricengine/archive.go:127-128`).
  - That tip is `job.WeftTip` (`internal/fabricengine/remoteteardown.go:127`), which `remoteteardown.go:70-72` describes as "the local weft tip the command tagged before it deleted the branch". It is empty only when no local tip exists (`remoteteardown.go:140`).
  - Card 7's remove row in `TestOneRepositoryPair_RecordsLiveInTheNamespace` removes a pair that has a local records branch, so the tip is set and `resolveArchiveTip` never runs.
  - A lookup left at `refs/heads/` there passes every planned row. This is the same gap the review raised for the four sites it named.
  - Fix: add a card 7 row that removes a pair with `--remote` whose local records branch is absent but whose branch is on origin, and assert the archive tag lands under `refs/lym/tags/archive/&lt;slug&gt;/`. Then add `resolveArchiveTip`'s remove path to the overview's list.

Every other member of the set is covered:
- Card 2: `takeoverBranch.land`, `pushClassified`, `pushJob.push`, `retryWeftPush`, `retryBoltPush`, `Fabric.PushWeft`, `archiveWeftTip`, `pushArchiveTag` and `deleteRemoteBranch`.
- Card 3: `remoteHeadTip`, `remoteTagsUnder`, `remoteBranchTips`, `remoteTagRefs`, `cleanupRemoteWarp`, `probeWeftLeftover`, `probeLiveLocalWeft`, `fetchMovedHeads`, `resolveWeftBranch`, `targetWeftTip`, `Topology.Add` and `readSideOwnerArchives`.
- Run only in the code store, so identity is correct: `repushOverOriginContained` (code-side retry, `pushclassified.go:235`), `pushOwedCodeTip`, and `updateRemoteBranch` at `pushclassified.go:302` and `destroy.go:2457`. The records-side `updateRemoteBranch` at `handover.go:259` falls under the hand-over row.

Tool calls:
1. Read card 3's Edit and Intent and card 2's fabricengine Edit list, to get the full set of changed sites; my context had neither card.
2. Grepped the callers of `originBranchHead`, `resolveArchiveTip`, `updateRemoteBranch`, `repushOverOriginContained` and `lsRemoteHeads`, to see where each runs.
3. Read the call contexts in `pushclassified.go`, `archive.go` and `destroy.go`, to tell records-side calls from code-side ones.
4. Read `awaitOrigin` and the callers of `archiveWeftTip`, to find when `observeOrigin` and an empty archive tip occur.
5. Grepped `WeftTip` in `remoteteardown.go`, to confirm it is empty only when no local tip exists.
