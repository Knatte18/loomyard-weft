```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. For each finding whose premise is a set, I reran its search and checked that the fix covers every member.

Findings left unchecked: F1 (the set of destructive-primitive calls outside `destroy.go` executors) and F13 (other card 3 Edit targets that change nothing). I checked F3, F4, F8 and F9.

1. **MEDIUM** — card 2's `pushJob.push` unseeded-board refusal (the fix for F4).
   The guard covers only one of the two board push sites, and the text doesn't say whether the refusal goes to the replay retry.
   Evidence:
   - `Bolt.PushRecorded` wires `retry := func() (bool, error) { return retryBoltPush(rec, b.path) }` (`internal/fabricengine/bolt.go:145`).
   - `retryBoltPush` pushes the board unleased with `repo.PushRebaseFree()` (`internal/fabricengine/pushclassified.go:878`).
   - `settleRejection` keeps retrying "while it fails with a bare rejection or an error no class names" (`pushclassified.go:182`).
   - So if the refusal is classified as a rejection, the replay could still push the unseeded board onto another hub's board, which is the case the guard exists to stop.
   The card should either say the refusal returns unclassified and is never retried, or put the same guard on `retryBoltPush`.

2. **MEDIUM** — the overview's coverage-of-namespace-sites decision (the fix for F8).
   It lists the sites card 7 reaches and names only `probeLiveLocalWeft` and `readSideOwnerArchives` as two-repository-only. That implies every other namespace site of cards 2 and 3 is reached, but some are not:
   - `Fabric.PushWeft`, a records push site in card 2's list, has no caller in card 7's flows. Its only caller in the tree is a test (`internal/fabricengine/mergesiblings_integration_test.go:208`).
   - `targetWeftTip` runs only from checkout (`internal/fabricengine/checkout.go:160`). Card 6's `Topology.Checkout` row reaches it, not card 7, and the decision doesn't mention checkout.
   A push site left at identity in `PushWeft` would still fail no test, which is the gap F8 named.

3. **LOW** — the same decision says "neither pushes to origin" about the two exempted readers. That's true, but `remoteTagsUnder` also runs from `destroy.go:1945` (the archive-coverage check) and from `remoteleftover.go:251`. The decision names neither, though their flows (remove, add-leftover) are reached. This is only a naming gap.

Checked with no finding:
- **F3:** no gitrepo test row in the changed cards spells the side token. The rows now use `x-records`/`y/x-records`; the other gitrepo rows use `<b>` or `main`.
- **F9:** every new production function now lands with a reaching caller.
  - Card 1: `remoteTrackingRef` is reached through `Upstream`.
  - Card 2: `Branch` through `pushPlainWithRetry`, `Tag` through `pushArchiveTag`, `LocalRef` through `namesRef`, and `RemoteFetchRefspecs` and `oneRepositoryNamespace` through `originNamespaceAt`, which the push sites call.
  - Card 4: `PushCloneBoard`, `AbandonSeededClone` and `pushCloneSeed` are reached through `CloneAndWire`.
  - The hubforge creates are exempt as test-support code.

Tool calls:
- grep for the callers of card 3's and card 2's namespace sites: my context didn't have their call graph.
- grep for the callers of `repushOverOriginContained` and `PushWeft`: the first grep found no plain-call caller for either.
- grep for the gitrepo push call sites in fabricengine: to list every board and records push site against the F4 guard and card 2's site list.
