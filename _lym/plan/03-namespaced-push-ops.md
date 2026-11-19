# Card 3 — namespaced-push-ops

**Edit:**
- `internal/gitrepo#Repo.Push` -> `func (r *Repo) Push(ns RefNamespace) error`
- `internal/gitrepo#Repo.pushWithRebaseRetry` -> `func (r *Repo) pushWithRebaseRetry(ns RefNamespace) error`
- `internal/gitrepo#Repo.pushPlainWithRetry` -> `func (r *Repo) pushPlainWithRetry(remote, source, branch string, ns RefNamespace) error`
- `internal/gitrepo#Repo.PushBranch` -> `func (r *Repo) PushBranch(remote, branch string, ns RefNamespace) error`
- `internal/gitrepo#Repo.PushCommit` -> `func (r *Repo) PushCommit(remote, sha, branch string, ns RefNamespace) error`
- `internal/gitrepo#Repo.PushRebaseFree` -> `func (r *Repo) PushRebaseFree(ns RefNamespace) error`
- `internal/gitrepo#Repo.PushCoalesced` -> `func (r *Repo) PushCoalesced(ns RefNamespace) error`
- `internal/gitrepo#Repo.DeleteRemoteBranch` -> `func (r *Repo) DeleteRemoteBranch(remote, branch string, ns RefNamespace) (deleted bool, err error)`
- `internal/gitrepo#Repo.DeleteRemoteBranchLeased` -> `func (r *Repo) DeleteRemoteBranchLeased(remote, branch, expectSHA string, ns RefNamespace) error`
- `internal/gitrepo#Repo.UpdateRemoteBranchLeased` -> `func (r *Repo) UpdateRemoteBranchLeased(remote, branch, sha, expectSHA string, ns RefNamespace) error`
- `internal/gitrepo#Repo.resolvePushDestination` -> `func (r *Repo) resolvePushDestination(remote, branch string, ns RefNamespace) (pushDestination, error)`
- `internal/gitrepo#pushDestination`
- `internal/gitrepo#Repo.recordPush`
- `internal/gitrepo#PushWithRetry` -> `func PushWithRetry(push func() error, ref string, ns RefNamespace, delays []time.Duration, sleep func(time.Duration)) error`
- `internal/gitrepo#PushWithRetryReasons` -> `func PushWithRetryReasons(push func() error, ref string, ns RefNamespace, delays []time.Duration, sleep func(time.Duration), extraReasons []string) error`
- `internal/gitrepo#retryablePushRefusal` -> `func retryablePushRefusal(err error, ref string, ns RefNamespace, extraReasons []string) (line string, retryable bool)`
- `internal/gitrepo#namesRef` -> `func namesRef(name, ref string, ns RefNamespace) bool`
- `internal/gitrepo/doc.go`
- `internal/gitrepo_test#TestDeleteRemoteBranch`
- `internal/gitrepo_test#TestFetch_RemoteAdvanced_UpdatesTrackingRefWithoutMovingHEAD`
- `internal/gitrepo_test#TestMergeResolveSHA`
- `internal/gitrepo_test#TestPull`
- `internal/gitrepo_test#newPushedMainClone`
- `internal/gitrepo_test#TestPushBranch`
- `internal/gitrepo_test#TestPush_AFailedRecordAfterALandedTransferStillSucceeds`
- `internal/gitrepo_test#TestPushBranch_APushWithNoTrackingRefToWriteLandsAndRecordsNothing`
- `internal/gitrepo_test#TestPushRebaseFree_ADefaultPushGoesToTheRemoteTheSettingsPick`
- `internal/gitrepo_test#TestPushLockPath_InGitDirAndCreatedThereByPushCoalesced`
- `internal/gitrepo#TestPlainPushPrimitives_ExhaustedRetryWrapsSentinelAndNeverRebases`
- `internal/gitrepo#TestPushWithRetry_RetriesARealRefusalOfATagPushedByItsFullRef`
- `internal/gitrepo#TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`
- `internal/gitrepo#TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`
- `internal/gitrepo_test#pushCoalescedChild`
- `internal/gitrepo_test#pushFromOtherClone`
- `internal/gitrepo_test#TestHasUnpulled`
- `internal/gitrepo_test#TestPush`
- `internal/gitrepo_test#TestPushCoalesced`
- `internal/gitrepo_test#TestUpdateRemoteBranchLeased`
- `internal/gitrepo_test#publishedClone`
- `internal/gitrepo_test#TestRemoteRefsLock_APushTransfersWhileTheLockIsHeldAndWritesItsTrackingRefOnceReleased`
- `internal/gitrepo_test#TestRemoteRefsLock_ATrackingRefAFetchMovedPastThePushedCommitStaysThere`
- `internal/gitrepo_test#TestRemoteRefsLock_ConcurrentFetchesAndPushesOfOneRepositoryAllSucceed`
- `internal/gitrepo_test#TestUpstreamSHA`
- `internal/fabricengine#takeoverBranch.land`
- `internal/fabricengine#pushJob.push`
- `internal/fabricengine#pushJob.repushOverOriginContained`
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#retryBoltPush`
- `internal/fabricengine#Fabric.PushWeft`
- `internal/fabricengine#archiveWeftTip`
- `internal/fabricengine#pushOwedCodeTip`
- `internal/fabricengine#pushArchiveTag`
- `internal/fabricengine#deleteRemoteBranch`
- `internal/fabricengine#updateRemoteBranch`
- `internal/fabricengine_test#TestCommitSeededStencils_ScopedCommitExcludesUnrelatedDirt`
- `internal/gitrepo_test#TestPush_NoRemoteConfigured_SurfacesGitError`

**Delete:**
- `internal/gitrepo#remoteRefAbsentTrigger`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#originNamespaceAt`
- `plan:internal/gitrepo#RefNamespace`

**Intent:**
Pass the remote ref namespace (card 1) to every `gitrepo` operation that sends or writes an origin ref name, as D2 states,
and have fabric hand each one the namespace of the repository it runs in.

`gitrepo`, each operation mapping branch and tag names through `ns`:
- `pushPlainWithRetry` pushes `source:` + `ns.Branch(branch)`; every plain push goes through it (`Push`, `PushBranch`, `PushCommit`, `PushRebaseFree`, `PushCoalesced` via `pushWithRebaseRetry`).
- `resolvePushDestination` matches the remote's fetch refspecs against `ns.Branch(branch)`, not `refs/heads/<branch>`, so the tracking ref of a namespaced push is the one the store's fetch refspec maps (`refs/remotes/origin/<b>` in both layouts).
  `pushDestination` gains a `namespace` field, and `recordPush` writes `branch.<b>.merge` as `dest.namespace.Branch(b)`: a one-repository records branch's upstream names `refs/lym/heads/<b>`.
- The leases read `--force-with-lease=` + `ns.Branch(branch)` + `:` + expect.
  `UpdateRemoteBranchLeased` accepts an empty `expectSHA` as a lease on absence (`--force-with-lease=<ref>:`), which card 5's seed push needs; a non-empty invalid SHA is still `ErrInvalidSHA`.
  A ref present on origin fails that lease as `ErrLeaseRejected`.
- `DeleteRemoteBranchLeased` deletes `ns.Branch(branch)` by its full name; an absent ref fails the lease as today.
- `DeleteRemoteBranch` first runs `git ls-remote <url> <ns.Branch(branch)>` (D4) and matches the exact ref line.
  An absent ref answers `(false, nil)` and pushes nothing; a present one is deleted by its full name.
  This holds in both layouts: git deletes a missing full-name ref silently, so the probe, not git's stderr, tells an absent branch from a removed one.
  The probe runs through `runChecked` in `DeleteRemoteBranch`'s own body, so `cmd/lym`'s `gitrepoPinnedRunBoundMethods` list stays as it is.
  Every production caller in fabric today passes a lease, so `TestDeleteRemoteBranch` is the probe's reaching test; D4 still requires it of the plain delete.
  `remoteRefAbsentTrigger` loses its use and is deleted.
- `PushWithRetry`, `PushWithRetryReasons`, `retryablePushRefusal` and `namesRef` take `ns`:
  a `! [remote rejected] <src> -> <dst> (<reason>)` line belongs to the pushed ref when `<src>` names it locally (short, `refs/heads/` or `refs/tags/` form) and `<dst>` names it locally or by `ns.Branch`/`ns.Tag`, which is how git prints a push into `refs/lym/`.

Fabric callers; a code-store site passes the zero namespace, every other site reads `originNamespaceAt` of the repository it pushes from and returns that read's error:
- records and board pushes: `pushJob.push` and `pushJob.repushOverOriginContained` (at `j.path`, any side, identity for the code side), `retryWeftPush`, `retryBoltPush`, `Fabric.PushWeft`, `takeoverBranch.land`, `archiveWeftTip`'s `PushCommit`;
- `pushArchiveTag` reads the namespace at `repoDir` and pushes `refs/tags/<tag>:` + `ns.Tag(tag)`, never a bare or source-only refspec, so a one-repository archive tag lands under `refs/lym/tags/archive/`;
- `deleteRemoteBranch` reads the namespace at `req.repoDir`, so a teardown deletes refs/lym/heads/<branch>-weft by its full name (D4);
- `pushOwedCodeTip` and `updateRemoteBranch` run in the code store and pass `gitrepo.RefNamespace{}`.

Every existing test call passes `gitrepo.RefNamespace{}` and keeps its assertion, with these changes and rows:
- `TestDeleteRemoteBranch`: its absent-ref row now asserts nothing was pushed (origin's refs unchanged, no tracking ref written);
  a new row deletes a namespaced `refs/lym/heads/<b>` on a bare, and one asserts a namespaced absent ref answers `(false, nil)`;
  one asserts an absent refs/lym/heads/x-weft answers `(false, nil)` while refs/lym/heads/y/x-weft, which shares its tail, exists and stays;
  a leased namespaced row deletes at the observed tip and fails `ErrLeaseRejected` once the ref moved.
  The namespaced rows read the bare with `gitkit.Git(t, container, "ls-remote", bare, "refs/lym/heads/*")`, since the shared `remoteHeads` reader runs `--heads`; that reader stays as it is.
- `TestCommitSeededStencils_ScopedCommitExcludesUnrelatedDirt` passes `gitrepo.RefNamespace{}` to its `PushCoalesced` call.
- `TestPush_NoRemoteConfigured_SurfacesGitError` wraps its `Push` and `PushCoalesced` method values in closures that pass `gitrepo.RefNamespace{}`, keeping its assertions.
- `TestUpdateRemoteBranchLeased`: rows for an empty expect landing on an absent ref, failing `ErrLeaseRejected` on a present one, and a namespaced move.
- `TestPushRebaseFree_ADefaultPushGoesToTheRemoteTheSettingsPick`: a row over a store with D1's fetch refspecs pushes to `refs/lym/heads/<b>`, writes `branch.<b>.merge` = `refs/lym/heads/<b>`, moves `refs/remotes/origin/<b>`, and `HasUnpushed` then answers false.
- `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`: a row whose destination is `refs/lym/heads/main` under the namespace is retryable, and the same line under the identity namespace is not.

`gitrepo/doc.go` rewrites its push and delete paragraphs for the namespace parameter and the probing plain delete;
fabric's `doc.go` gains a paragraph naming which push sites read the namespace.
Fabric's behaviour in a one-repository hub is covered by card 9's tests, since no such hub exists before card 7.

**ImpactSummary:** Re-signs ten exported gitrepo push/lease/delete operations; every caller is in fabricengine (eleven members listed here), one fabricengine_test test or a gitrepo test, and two-repository behaviour is unchanged except that a plain delete now probes with ls-remote first.
