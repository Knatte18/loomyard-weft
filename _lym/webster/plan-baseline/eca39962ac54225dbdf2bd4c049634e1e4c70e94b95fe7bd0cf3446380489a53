# Card 7 — pair-lifecycle-namespace

**Create:**
- `internal/fabricengine_test#TestOneRepositoryPair_RecordsLiveInTheNamespace`
- `internal/fabricengine_test#TestOneRepositoryOrigin_ReconcileAndCleanupSeeTheNamespace`
- `internal/fabricengine_test#TestOneRepositorySecondMachine_AdoptPullAndReplay`
- `internal/fabricengine_test#TestOneRepositoryMerge_LeavesNoRecordsOnTheCodeBranch`

**Edit:**
- `internal/fabricengine_test#TestCleanup_NeverPushedOrphanIsIdempotentOnRemote`

**Uses:**
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPair`
- `internal/hubforge#AddPairWith`
- `internal/hubforge#OpenFabric`
- `internal/hubforge#CloneSecond`
- `internal/fabricengine#Fabric.PushBranch`
- `internal/fabricengine#Fabric.Pull`
- `internal/fabricengine#Bolt.Push`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#Topology.Cleanup`
- `internal/fabricengine#Topology.CleanupRemoteWarp`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#TakeOverRun`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine#ArchiveOwnerTips`
- `internal/fabricengine#Topology.ReadOwnerArchives`
- `internal/fabricengine#Fabric.PushWeft`
- `internal/fabricengine#ArchiveWeftTipForTest`

**Intent:**
Pin D2 and D4 in a one-repository hub with the two-repository suites' key flows over card 5's builder (D8).
These tests reach the one-repository sites the overview's coverage-of-namespace-sites decision names. Every test is tier1, in `fabricengine_test`, and reads the bare with `git for-each-ref`.

- `TestOneRepositoryPair_RecordsLiveInTheNamespace` (D2, D4):
  `AddPair` then `Fabric.PushBranch` place the pair's records branch at refs/lym/heads/<branch>-weft;
  the records branch's `branch.<b>.merge` is refs/lym/heads/<branch>-weft and `HasUnpushed` answers false right after the push;
  the only ref the pair adds outside `refs/lym/` is its code branch under `refs/heads/`, and none under `refs/tags/`;
  `Topology.Remove` with its remote step puts the archive tag at `refs/lym/tags/archive/<slug>/<tip>` and deletes refs/lym/heads/<branch>-weft;
  a second row has `Topology.Add` of a removed slug whose namespaced records branch its archive tag covers reach `probeWeftLeftover`, which replaces that leftover;
  a third row pushes a records branch, deletes it locally, and calls `ArchiveWeftTipForTest`, which reads origin's tip through `resolveArchiveTip`
  and puts the archive tag at `refs/lym/tags/archive/<slug>/<tip>` at refs/lym/heads/<branch>-weft's tip.
- `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote` (D4, "in either layout") becomes a table over two builders, its body the shared row:
  the existing row over `CopyHub`, and a row over `NewOneRepoHub`; in each, a never-pushed records branch swept with `--remote` records no `remote_branch_deleted` entry and leaves origin's refs as they were.
  The one-repository row first creates a code branch under `refs/heads/` on the bare with the swept records branch's name,
  and asserts it survives the `--remote` sweep at its tip with no `remote_branch_deleted` entry, so a tip read that ignores the namespace fails the row.
  Its `//testtiming:keep` reason still holds.
- `TestOneRepositoryOrigin_ReconcileAndCleanupSeeTheNamespace` (D2, D4):
  a pair pushed from a `CloneSecond` machine is seen by `ReconcileOrigin` on the first, which fetches its records branch to refs/remotes/origin/<branch>-weft;
  `Topology.CleanupRemoteWarp` dry run, whose records-store listing reads `remoteBranchTips` and `remoteTagRefs` of the records store,
  classifies the code branches by the namespaced records branches and archive tags and reports no entry for a records branch as a code branch;
  with `apply`, origin's code `refs/heads/main` survives, as does every `refs/lym/` ref.
- `TestOneRepositorySecondMachine_AdoptPullAndReplay` (D4: add from origin, pull, replay, checkout, records push, owner archives), over `NewOneRepoHub` and its `CloneSecond`:
  - adopt: the first hub adds and pushes a pair; `Topology.Add` of the same slug on the second adopts its records branch from refs/lym/heads/<branch>-weft, with the first hub's records commits;
  - pull: the first hub commits and pushes more records on that pair; `Fabric.Pull` on the second brings them in, and its records branch's upstream reads in sync;
  - replay: both hubs commit to their boards; the first pushes with `Bolt.Push`, the second's push is rejected and replays, and `refs/lym/heads/main` then holds both commits;
  - records replay: both hubs commit records on the adopted pair; the first pushes, the second's `Fabric.PushBranch` is rejected and lands through the records replay (`retryWeftPush`),
    and refs/lym/heads/<branch>-weft then holds both commits;
  - hand-over: the second machine takes over the first's run with `TakeOverRun`; its records land at refs/lym/heads/<branch>-weft,
    and nothing new appears under `refs/heads/` but the code branch;
  - live local branch, on a slug of its own: the first hub adds and pushes a pair; the second machine adopts it,
    removes it with `Topology.Remove` and no remote step, and recreates its records branch at the removed tip with `git branch`;
    the first hub then pushes one more records commit, so origin's refs/lym/heads/<branch>-weft is ahead of the second machine's branch;
    `Topology.Add` of the slug on the second machine reaches `probeLiveLocalWeft`, records a `KindRepoAdvanced` mutation naming origin's tip,
    and leaves the adopted records worktree holding the first hub's later commit;
  - checkout: the second machine pushes a pair whose records branch the first hub holds nowhere locally;
    after a fetch of its code store, the first hub's `Topology.Checkout` of the prime to that branch reaches `targetWeftTip`,
    which fetches refs/lym/heads/<branch>-weft into refs/remotes/origin/<branch>-weft,
    and the records worktree ends on that branch with the second machine's commits;
  - records push: `Fabric.PushWeft` on a hub holding an unpushed records commit lands it at refs/lym/heads/<branch>-weft and adds no ref under `refs/heads/`;
  - owner archives: `ArchiveOwnerTips` on the first hub for a pair holding an unpushed records commit pushes its owner archive tag under `refs/lym/tags/`,
    and `Topology.ReadOwnerArchives` on the second machine lists it on the records side and fetches it as a local tag.
- `TestOneRepositoryMerge_LeavesNoRecordsOnTheCodeBranch` (D2, D5): a pair's work merged by fabric into the prime's code branch leaves no `_lym` path in that branch's tree and no `Fabric-Merge` trailer in the merge commit.

**ImpactSummary:** Test-only: one existing test becomes a two-row table with its assertions kept and its one-repository row guarded by a same-named code branch, and four new tests read the namespace; no production code changes.

**Commit:** `7: pin the pair lifecycle in the one-repository namespace`
