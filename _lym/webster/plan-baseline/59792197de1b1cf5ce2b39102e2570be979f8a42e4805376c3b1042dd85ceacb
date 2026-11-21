# Card 2 — namespaced-push-ops

**Create:**
- `plan:internal/gitrepo#RefNamespace` -> `type RefNamespace struct`
- `plan:internal/gitrepo#RefNamespace.Branch` -> `func (n RefNamespace) Branch(name string) string`
- `plan:internal/gitrepo#RefNamespace.Tag` -> `func (n RefNamespace) Tag(name string) string`
- `plan:internal/gitrepo#RefNamespace.LocalRef` -> `func (n RefNamespace) LocalRef(ref string) (string, bool)`
- `plan:internal/gitrepo#Repo.RemoteFetchRefspecs` -> `func (r *Repo) RemoteFetchRefspecs(name string) ([]string, error)`
- `plan:internal/gitrepo_test#TestRefNamespace` -> `func TestRefNamespace(t *testing.T)`
- `plan:internal/gitrepo_test#TestRemoteFetchRefspecs` -> `func TestRemoteFetchRefspecs(t *testing.T)`
- `plan:internal/fabricengine#RecordsNamespaceRoot` -> `const RecordsNamespaceRoot = "refs/lym/"`
- `plan:internal/fabricengine#RecordsNamespaceHeads` -> `const RecordsNamespaceHeads = "heads/"`
- `plan:internal/fabricengine#RecordsNamespaceTags` -> `const RecordsNamespaceTags = "tags/"`
- `plan:internal/fabricengine#OneRepositoryBoardBranch` -> `const OneRepositoryBoardBranch = "main"`
- `plan:internal/fabricengine#PrimeRecordsBranchKey` -> `const PrimeRecordsBranchKey = "lym.primeRecordsBranch"`
- `plan:internal/fabricengine#oneRepositoryNamespace` -> `func oneRepositoryNamespace() gitrepo.RefNamespace`
- `plan:internal/fabricengine#namespaceAnswers` -> `var namespaceAnswers sync.Map`
- `plan:internal/fabricengine#originNamespaceAt` -> `func originNamespaceAt(dir string) (gitrepo.RefNamespace, error)`
- `plan:internal/fabricengine#TestOriginNamespaceAt` -> `func TestOriginNamespaceAt(t *testing.T)`

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
- `internal/fabricengine#pushClassified`
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

**Intent:**
Add `gitrepo`'s remote ref namespace and fabric's layout read (D1, D2), pass the namespace to every `gitrepo` operation that sends or writes an origin ref name, as D2 states,
and have fabric hand each one the namespace of the repository it runs in.
The namespace type, the refspec read and the layout read land here, beside their first production callers, so no batch boundary leaves them unreached from a `main`.

**The namespace type.**
`RefNamespace` has two string fields, `Heads` and `Tags`, the origin-side prefixes of branches and tags.
Its zero value is the identity namespace of a two-repository hub, `refs/heads/` and `refs/tags/`.
Fabric builds the one-repository value from its own constants; `gitrepo` holds no `refs/lym/` literal and learns no layout: it receives a namespace and maps names through it.
- `Branch(name)` returns the origin ref of a branch: `Heads+name`, or `refs/heads/<name>` for an empty `Heads`.
- `Tag(name)` does the same for a tag.
- `LocalRef(ref)` maps an origin ref name read back from `ls-remote` or a push result to its local `refs/heads/` or `refs/tags/` name.
  A ref outside both prefixes answers false, so a one-repository listing never reads a code ref as a records ref.

`Repo.RemoteFetchRefspecs(name)` returns the configured `remote.<name>.fetch` refspecs, read through go-git's local config like `RemoteURL`, spawning no git.
A remote that is not configured returns an error wrapping `ErrRemoteNotConfigured`; any other config read failure is returned as it is.
Fabric's layout read is its one caller.

**The layout constants and read**, in a new file of `internal/fabricengine`.
Constants, the only spelling of the layout in the tree:
`RecordsNamespaceRoot` (`refs/lym/`), `RecordsNamespaceHeads` (`heads/`), `RecordsNamespaceTags` (`tags/`),
`OneRepositoryBoardBranch` (`main`, the board's fixed branch) and `PrimeRecordsBranchKey` (`lym.primeRecordsBranch`).
`OneRepositoryBoardBranch` is a branch lym owns inside its records namespace, which the declared `PATTERN-code-conventions` change admits as one named constant in fabric (card 8).
`oneRepositoryNamespace()` returns `gitrepo.RefNamespace{Heads: RecordsNamespaceRoot+RecordsNamespaceHeads, Tags: RecordsNamespaceRoot+RecordsNamespaceTags}`.

`originNamespaceAt(dir)` returns the remote ref namespace of the repository whose worktree is at `dir`, read from its `origin` fetch refspecs:
- a refspec whose source side starts with `refs/lym/heads/` answers `oneRepositoryNamespace()`;
- otherwise the identity namespace, the zero value.
  A code store keeps git's default refspecs, so the read answers identity there and fabric's translation never touches it.
- `dir` that does not exist, or holds no `.git` entry, answers the identity namespace, logs nothing and is not cached,
  so a store created there later is read then.
- A repository with no `origin` remote answers identity and is cached; such a store pushes nothing.
- Any other failure of the config read is returned as an error, never an identity answer, and is not cached.
- A successful read of an existing store is cached in `namespaceAnswers`, keyed by the cleaned path, for the process;
  a later call for the same path runs no git and opens no config.
  The answer cannot change for a store, since only a clone writes the refspecs, before the store's first read.

**`gitrepo`'s operations**, each mapping branch and tag names through `ns`:
- `pushPlainWithRetry` pushes `source:` + `ns.Branch(branch)`; every plain push goes through it (`Push`, `PushBranch`, `PushCommit`, `PushRebaseFree`, `PushCoalesced` via `pushWithRebaseRetry`).
- `resolvePushDestination` matches the remote's fetch refspecs against `ns.Branch(branch)`, not `refs/heads/<branch>`, so the tracking ref of a namespaced push is the one the store's fetch refspec maps (`refs/remotes/origin/<b>` in both layouts).
  `pushDestination` gains a `namespace` field, and `recordPush` writes `branch.<b>.merge` as `dest.namespace.Branch(b)`: a one-repository records branch's upstream names `refs/lym/heads/<b>`.
- The leases read `--force-with-lease=` + `ns.Branch(branch)` + `:` + expect.
  `UpdateRemoteBranchLeased` accepts an empty `expectSHA` as a lease on absence (`--force-with-lease=<ref>:`), which card 4's seed push needs; a non-empty invalid SHA is still `ErrInvalidSHA`.
  A ref present on origin fails that lease as `ErrLeaseRejected`.
- `DeleteRemoteBranchLeased` deletes `ns.Branch(branch)` by its full name; an absent ref fails the lease as today.
- `DeleteRemoteBranch` first runs `git ls-remote <url> <ns.Branch(branch)>` (D4) and matches the exact ref line.
  An absent ref answers `(false, nil)` and pushes nothing; a present one is deleted by its full name.
  This holds in both layouts: git deletes a missing full-name ref silently, so the probe, not git's stderr, tells an absent branch from a removed one.
  The probe runs through `runChecked` in `DeleteRemoteBranch`'s own body, so `cmd/lym`'s `gitrepoPinnedRunBoundMethods` list stays as it is.
  Every production caller in fabric today passes a lease, so `TestDeleteRemoteBranch` is the probe's reaching test; D4 still requires it of the plain delete.
  `remoteRefAbsentTrigger` loses its use and is deleted.
- `PushWithRetry`, `PushWithRetryReasons`, `retryablePushRefusal` and `namesRef` take `ns`:
  a `! [remote rejected] <src> -> <dst> (<reason>)` line belongs to the pushed ref when `<src>` names it locally (short, `refs/heads/` or `refs/tags/` form)
  and `<dst>` names it locally or maps through `ns.LocalRef` to its local name, which is how git prints a push into `refs/lym/`.

**Fabric callers**; a code-store site passes the zero namespace, every other site reads `originNamespaceAt` of the repository it pushes from and returns that read's error:
- records and board pushes: `pushJob.push` and `pushJob.repushOverOriginContained` (at `j.path`, any side, identity for the code side), `retryWeftPush`, `retryBoltPush`, `Fabric.PushWeft`, `takeoverBranch.land`, `archiveWeftTip`'s `PushCommit`;
- `pushClassified` refuses, for side `bolt`, a board whose branch has no upstream in a records store whose `originNamespaceAt` reads as one repository,
  before its upstream read and before any push:
  the only such board is a `--single-repo` seed that `PushCloneBoard` never pushed, since a seeded board takes its upstream at the seed push
  and a joined board adopts `origin/main` through `ensureBoardWorktree`'s `--track` form.
  The refusal names one way forward: run the same `lym fabric clone --single-repo` again with `--reset`.
  It is returned as it is: no failed-push record is written, so no later push or reconcile retries it, and no replay runs, so `retryBoltPush`, the board's one replay, never pushes that board.
  It keeps `lym fabric reconcile` and every other board push from pushing an unseeded board unleased, and from replaying it onto another hub's board (D3's bound);
- `pushArchiveTag` reads the namespace at `repoDir` and pushes `refs/tags/<tag>:` + `ns.Tag(tag)`, never a bare or source-only refspec, so a one-repository archive tag lands under `refs/lym/tags/archive/`;
- `deleteRemoteBranch` reads the namespace at `req.repoDir`, so a teardown deletes refs/lym/heads/<branch>-weft by its full name (D4);
- `pushOwedCodeTip` and `updateRemoteBranch` run in the code store and pass `gitrepo.RefNamespace{}`.

Tests:
- `TestRefNamespace` (tier0) goes in a new tier0-only test file of `internal/gitrepo`, which spawns no git.
  It is a table over the identity and a namespaced value: `Branch`, `Tag`, and `LocalRef` of a namespaced head, a namespaced tag, and a ref outside both, which answers false.
- `TestRemoteFetchRefspecs` (tier1) goes in `remote_integration_test.go`, the existing tier1 file of the `RemoteURL` tests.
  Rows: a default clone answers `+refs/heads/*:refs/remotes/origin/*`;
  a store configured with `+refs/lym/heads/*:refs/remotes/origin/*` and `+refs/lym/tags/*:refs/tags/*` answers both;
  an unconfigured remote name returns an error matching `ErrRemoteNotConfigured`.
- `TestOriginNamespaceAt` (tier1, internal test; it swaps the process-global log output and reads the process-wide cache, so it is not parallel) rows:
  - a missing path and a directory without `.git` answer identity and log nothing;
  - a store cloned with default refspecs answers identity;
  - a store configured with `+refs/lym/heads/*:refs/remotes/origin/*` answers `oneRepositoryNamespace()`;
  - a second read of that store after its fetch refspec is rewritten behind fabric's back still answers the namespaced value, showing the read was not repeated;
  - a path read while missing, then created as a namespaced store, answers the namespaced value;
  - a repository with no `origin` remote answers identity, and a second read after an `origin` with the namespaced refspec is added still answers identity, showing the answer was cached;
  - a store whose `.git` is an empty file answers an error; once it is replaced by a namespaced store, the next read answers the namespaced value, showing the failed read was not cached.

Every existing test call passes `gitrepo.RefNamespace{}` and keeps its assertion, with these changes and rows:
- `TestDeleteRemoteBranch`: its absent-ref row now asserts nothing was pushed (origin's refs unchanged, no tracking ref written);
  a new row deletes a namespaced `refs/lym/heads/<b>` on a bare, and one asserts a namespaced absent ref answers `(false, nil)`;
  one asserts an absent refs/lym/heads/x-records answers `(false, nil)` while refs/lym/heads/y/x-records, which shares its tail, exists and stays;
  a leased namespaced row deletes at the observed tip and fails `ErrLeaseRejected` once the ref moved.
  The namespaced rows read the bare with `gitkit.Git(t, container, "ls-remote", bare, "refs/lym/heads/*")`, since the shared `remoteHeads` reader runs `--heads`; that reader stays as it is.
- `TestCommitSeededStencils_ScopedCommitExcludesUnrelatedDirt` passes `gitrepo.RefNamespace{}` to its `PushCoalesced` call.
- `TestPush_NoRemoteConfigured_SurfacesGitError` wraps its `Push` and `PushCoalesced` method values in closures that pass `gitrepo.RefNamespace{}`, keeping its assertions.
- `TestUpdateRemoteBranchLeased`: rows for an empty expect landing on an absent ref, failing `ErrLeaseRejected` on a present one, and a namespaced move.
- `TestPushRebaseFree_ADefaultPushGoesToTheRemoteTheSettingsPick`: a row over a store with D1's fetch refspecs pushes to `refs/lym/heads/<b>`, writes `branch.<b>.merge` = `refs/lym/heads/<b>`, moves `refs/remotes/origin/<b>`, and `HasUnpushed` then answers false.
- `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`: a row whose destination is `refs/lym/heads/main` under the namespace is retryable, and the same line under the identity namespace is not.

`gitrepo/doc.go` gains a paragraph on `RefNamespace` and rewrites its push and delete paragraphs for the namespace parameter and the probing plain delete;
fabric's `doc.go` gains the layout section (the constants, the refspec as the layout, the cached quiet read) and a paragraph naming which push sites read the namespace and the unseeded-board refusal.
Fabric's behaviour in a one-repository hub is covered by card 7's tests, since no such hub exists before card 5; card 5's seed-failure test reaches the unseeded-board refusal.

**ImpactSummary:** Adds gitrepo's namespace type and refspec read and fabric's layout constants and cached read, each reached through the push sites changed here; re-signs ten exported gitrepo push/lease/delete operations, whose callers are all in fabricengine (eleven members listed here), one fabricengine_test test or a gitrepo test; two-repository behaviour is unchanged except that a plain delete now probes with ls-remote first.
