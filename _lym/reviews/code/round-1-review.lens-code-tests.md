```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests — per behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

- RefNamespace.Branch/Tag/LocalRef mapping: `gitrepo.TestRefNamespace` (refnamespace_test.go). Pinned.
- Push/PushBranch/PushCommit/PushRebaseFree to a namespaced destination, with tracking ref and `branch.<b>.merge`: `TestPushRebaseFree_ADefaultPushGoesToTheRemoteTheSettingsPick/NamespacedFetchRefspecs`. Pinned for PushRebaseFree only. `PushBranch` and `PushCommit` have no namespaced gitrepo row, but the one-repository lifecycle tests reach them (archive of an origin-only branch, PushBranch in the pair test). Pinned indirectly.
- UpdateRemoteBranchLeased, empty lease on absence, and namespaced create/move: `TestUpdateRemoteBranchLeased` rows. Pinned.
- DeleteRemoteBranch, ls-remote first, absent ref pushes nothing, namespaced, tail-sharing ref kept: `TestDeleteRemoteBranch` rows. Pinned.
- DeleteRemoteBranchLeased, namespaced lease at stale and at observed tip: `TestDeleteRemoteBranch` row. Pinned.
- Push-refusal retry matching through a namespace (`namesRef`/`retryablePushRefusal`): `TestRetryablePushRefusal_*` namespaced rows. Pinned for branches. The archive-tag path with a namespaced tag destination (`refs/tags/x` source against a `refs/lym/tags/x` destination) has no row. This is a gap.
- `PushWithRetryReasons` called with a namespace: reached by the archive-tag test, but the retry itself is never exercised under a namespace. This is the same gap.
- `remoteTrackingRef`/`Upstream`/`UpstreamSHA` following refspecs, and no upstream for an unmapped merge ref: `TestParity` namespaced row and `TestUpstreamSHA` refspec rows. Pinned.
- `RemoteFetchRefspecs`: `TestRemoteFetchRefspecs`. Pinned.
- Tolerant go-git config open (`openGoGitWithUnvalidatedBranchConfig`, `decodeConfigWithoutBranchValidation`): reached by the parity and namespaced upstream rows, which would fail on a refused config. Linked worktree with the new open: `TestLinkedWorktree_Parity` (existing). The `.git` file with a relative gitdir and a relative commondir are not distinctly pinned. The `ErrRepositoryNotExists` path for a non-repo path is covered by the existing `TestGoGit_NonRepoPath_*`. The empty-config read (`os.IsNotExist` → empty config) has no row, and it is minor.
- `originNamespaceAt` (missing store quiet and uncached, cached read, error not cached, no-origin identity): `TestOriginNamespaceAt`. Pinned. The "second read runs no git" criterion is checked only through a changed config that is not re-read; that is acceptable.
- `readOriginLayout`/`refuseOneURLClone` refusals: `TestCloneHub_OneURLRefusals`. Rows cover lym-bolt with and without the flag, board present, records repo, legacy anchor, `_lym` tracked, no shortname and second URL. Missing rows:
  - The criterion "a two-repository weft whose board tracks the legacy `.lym-anchor` clones as today" is pinned by `TestCloneHub_AnchorAdoptPath/BoltMarkerOneURL`. Pinned.
  - The CLI-level "second URL" refusal is pinned in `cli_refusals_test.go`.
  - The `--single-repo` with `--reset` clean path has no row.
- Seed push order, leases, teardown, restore: `TestCloneSingleRepo_FailedSeedPushLeavesNothing`. Rows cover a refused board, leftovers under the same and another subpath, records lease lost, board lease lost (absent and leftover observed), restore failure, a transient lock, abandon and unseeded push. Pinned. The order itself (records before board) is pinned by "BoardLeaseLost" leaving the records restored; a swapped order would also fail `RecordsLeaseLost`.
- Seed namespace placement, `master` default, code refs untouched, default `git clone` fetches no `lym` refs: `TestCloneSingleRepo_SeedsTheNamespace`. Pinned.
- Join, second machine, `master` default, shortname/binding mismatch, `--reset` subpath handling: `TestCloneOneURL_JoinsTheNamespacedHub`. Pinned.
- `AbandonSeededClone` and `PushCloneBoard` (new): the abandon rows, plus `TestCommand_CloneEndToEnd` through the reach report. Pinned.
- `refuseUnseededBoard`: the "UnseededBoardIsNeverPushed" row. Pinned.
- `primaryWeftBranch` key read, fail closed, kept across checkout: `TestCleanup_ProtectsPrimaryWeftBranchAfterCheckout` and `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable`, both over both layouts. Pinned. The `master` + checkout-off case is covered by the OneRepository row.
- Plain remote delete of a never-pushed branch records no removal, in both layouts: `TestCleanup_NeverPushedOrphanIsIdempotentOnRemote` over both layouts, plus the gitrepo row. Pinned.
- Pair lifecycle (push, archive tag, remove, add leftover, origin-only archive, reconcile, remote cleanup, pull, replays, takeover, live-local probe, checkout fetch, `PushWeft`, owner archives, merge): `onerepository_lifecycle_integration_test.go`. Pinned.
- `checkSubpathBeforeReset` / `recordedAnchorOnWeft` under a namespace: the reset subpath rows in the join test. Pinned.
- `remoteBranchTips`/`remoteTagRefs` mapping, including the peeled `^{}` suffix handling in `remoteTagRefs`: the remote-cleanup test classifies by namespaced archive tags. It uses lightweight tags only, so the `^{}` re-append is not exercised. This is the same gap in the annotated-tag path of `remoteTagsUnder`, which is also covered only with lightweight tags. Low.
- hubforge `CopyCodeBare`/`NewOneRepoHub`: used by every row above. Pinned.
- docs, PATTERN: static prose, exempt.

**Findings**

1. LOW, class `design`. Location: `internal/gitrepo/pushretry_test.go` (namespaced rows at about line 66), against `internal/gitrepo/pushretry.go` `namesRef`/`namesLocalRef`. Summary: no row pins the namespaced tag path of the push-refusal retry. A source `refs/tags/<t>` with destination `refs/lym/tags/<t>` is what `pushArchiveTag` sends in a one-repository hub, and every namespaced row uses a branch. Fix: add a row `{ref: "refs/tags/archive/s/abc", line: " ! [remote rejected] refs/tags/archive/s/abc -> refs/lym/tags/archive/s/abc (failed)", ns: namespaced, want: true}` and one under the identity namespace with `want: false`.

2. LOW, class `design`. Location: `internal/fabricengine/onerepository_lifecycle_integration_test.go` (`CleanupRemoteWarpClassifiesByTheNamespacedEvidence`, `OwnerArchivesAreTaggedInTheNamespaceAndReadBackBySecondMachine`), against `remoteTagRefs` in `cleanupremotewarp.go` and `remoteTagsUnder` in `remoteleftover.go`. Summary: the peeled `^{}` handling the diff rewrote in both readers is never exercised. Every archive tag in these tests is lightweight, and annotated archive tags are what the fabric flows create. Fix: make one archive tag in the cleanup test annotated (`git tag -a`) and assert the classification and the tip still resolve to the commit.

No other behaviour lacks a covering row.

**Tool calls:** none made. The context already held every diff and the reach report, so no extra reads were needed.
