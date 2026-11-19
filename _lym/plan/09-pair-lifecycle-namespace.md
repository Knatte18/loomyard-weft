# Card 9 — pair-lifecycle-namespace

**Create:**
- `plan:internal/fabricengine_test#TestOneRepositoryPair_RecordsLiveInTheNamespace` -> `func TestOneRepositoryPair_RecordsLiveInTheNamespace(t *testing.T)`
- `plan:internal/fabricengine_test#TestOneRepositoryOrigin_ReconcileAndCleanupSeeTheNamespace` -> `func TestOneRepositoryOrigin_ReconcileAndCleanupSeeTheNamespace(t *testing.T)`
- `plan:internal/fabricengine_test#TestOneRepositorySecondMachine_AdoptPullAndReplay` -> `func TestOneRepositorySecondMachine_AdoptPullAndReplay(t *testing.T)`
- `plan:internal/fabricengine_test#TestOneRepositoryMerge_LeavesNoRecordsOnTheCodeBranch` -> `func TestOneRepositoryMerge_LeavesNoRecordsOnTheCodeBranch(t *testing.T)`

**Edit:**
- `internal/fabricengine_test#TestCleanup_NeverPushedOrphanIsIdempotentOnRemote`

**Uses:**
- `plan:internal/hubforge#NewOneRepoHub`
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

**Intent:**
Pin D2 and D4 in a one-repository hub with the two-repository suites' key flows over card 7's builder (D8).
These tests reach the namespace paths cards 3 and 4 changed. Every test is tier1, in `fabricengine_test`, and reads the bare with `git for-each-ref`.

- `TestOneRepositoryPair_RecordsLiveInTheNamespace` (D2, D4):
  `AddPair` then `Fabric.PushBranch` place the pair's records branch at refs/lym/heads/<branch>-weft;
  the records branch's `branch.<b>.merge` is refs/lym/heads/<branch>-weft and `HasUnpushed` answers false right after the push;
  the only ref the pair adds outside `refs/lym/` is its code branch under `refs/heads/`, and none under `refs/tags/`;
  `Topology.Remove` with its remote step puts the archive tag at `refs/lym/tags/archive/<slug>/<tip>` and deletes refs/lym/heads/<branch>-weft.
- `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote` (D4, "in either layout") becomes a table over two builders, its body the shared row:
  the existing row over `CopyHub`, and a row over `NewOneRepoHub`; in each, a never-pushed records branch swept with `--remote` records no `remote_branch_deleted` entry and leaves origin's refs as they were.
  Its `//testtiming:keep` reason still holds.
- `TestOneRepositoryOrigin_ReconcileAndCleanupSeeTheNamespace` (D2, D4):
  a pair pushed from a `CloneSecond` machine is seen by `ReconcileOrigin` on the first, which fetches its records branch to refs/remotes/origin/<branch>-weft;
  `Topology.CleanupRemoteWarp` dry run, whose records-store listing reads `remoteBranchTips` and `remoteTagRefs` of the records store,
  classifies the code branches by the namespaced records branches and archive tags and reports no entry for a records branch as a code branch;
  with `apply`, origin's code `refs/heads/main` survives, as does every `refs/lym/` ref.
- `TestOneRepositorySecondMachine_AdoptPullAndReplay` (D4: add from origin, pull, replay), over `NewOneRepoHub` and its `CloneSecond`:
  - adopt: the first hub adds and pushes a pair; `Topology.Add` of the same slug on the second adopts its records branch from refs/lym/heads/<branch>-weft, with the first hub's records commits;
  - pull: the first hub commits and pushes more records on that pair; `Fabric.Pull` on the second brings them in, and its records branch's upstream reads in sync;
  - replay: both hubs commit to their boards; the first pushes with `Bolt.Push`, the second's push is rejected and replays, and `refs/lym/heads/main` then holds both commits.
  The hand-over's records-side reads are the `Fetch` and upstream read the pull row reaches.
- `TestOneRepositoryMerge_LeavesNoRecordsOnTheCodeBranch` (D2, D5): a pair's work merged by fabric into the prime's code branch leaves no `_lym` path in that branch's tree and no `Fabric-Merge` trailer in the merge commit.

**ImpactSummary:** Test-only: one existing test becomes a two-row table with its assertions kept, and four new tests read the namespace; no production code changes.

**Commit:** `9: pin the pair lifecycle in the one-repository namespace`
