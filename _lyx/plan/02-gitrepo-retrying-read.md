# Card 2 — Every go-git read retries whole after a repack

**Create:**
- `plan:internal/gitrepo#readGoGit` -> `func readGoGit[T any](r *Repo, read func(repo *git.Repository) (T, error)) (T, error)`
- `internal/gitrepo/gogitcaller_test.go`
- `pattern/PATTERN-gogit-read-helper.md`

**Edit:**
- `internal/gitrepo#Repo`
- `internal/gitrepo#Repo.goGit`
- `internal/gitrepo#Repo.CurrentSHA`
- `internal/gitrepo#Repo.SHAExists`
- `internal/gitrepo#Repo.CurrentBranch`
- `internal/gitrepo#Repo.ChangedFilesSince`
- `internal/gitrepo#Repo.WorktreeChangedFiles`
- `internal/gitrepo#Repo.UntrackedFiles`
- `internal/gitrepo#Repo.RemoteURL`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/gitrepo#Repo.HeadContains`
- `internal/gitrepo#Repo.PathRevisions`
- `internal/gitrepo#Repo.FilesInDirAtRevision`
- `internal/gitrepo#Repo.CommitsWithSubject`
- `internal/gitrepo#Repo.HeadDetached`
- `internal/gitrepo#Repo.CommitParents`
- `internal/gitrepo#Repo.ResolveSHA`
- `internal/gitrepo#Repo.CommitTree`
- `internal/gitrepo/doc.go`
- `internal/gitrepo/gogit_test.go`
- `internal/gitrepo/parity_test.go`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/gitrepo#lookupObjectRetrying`

**Uses:**
- `internal/gitrepo#packFingerprint`
- `internal/pattern#MaxOverviewBytes`

**Intent:** Decision D1 (#468): a cached go-git handle reads a stale pack index after a repack, and `Worktree.Status()`, `Log` walks and tree diffs bypass the per-lookup `lookupObjectRetrying`, so they report `object not found`.
Replace `lookupObjectRetrying` with one helper, `readGoGit`, in `gogit.go`, that runs a caller's whole go-git read and retries the whole read once.

`readGoGit`'s discipline, since `r.goGitMu` is a non-reentrant `sync.RWMutex`:

- It resolves the handle through `goGit()` before taking any lock, and is `goGit()`'s only production caller.
- Under the shared lock it snapshots `r.lastPackFingerprint`, then runs `read(repo)`.
- On an error whose chain holds `plumbing.ErrObjectNotFound`, it releases the shared lock and takes the exclusive lock around the fingerprint check alone.
  It computes the current fingerprint with `packFingerprint`: equal to the snapshot, it returns the read's error, since the pack set has not changed since the read began; otherwise it calls `storer.Reindex()` and stores the new fingerprint only when the current one also differs from `r.lastPackFingerprint`, and reruns `read` under the shared lock either way.
  A concurrent caller's reindex in the gap between the two locks therefore still triggers this caller's rerun.
  This is the check for the carried-over `d1-split-lock-retry-race` finding: comparing against the snapshot, never against `lastPackFingerprint` alone, is what closes it.
- A storer that is not `*filesystem.Storage`, or a fingerprint read error, returns the read's error unchanged.
- It retries at most once per call, and only on object-not-found; every other error passes through unchanged.

Add an unexported reindex counter field to `Repo`, incremented under the exclusive lock beside each `Reindex()`, read only by the package's internal tests.
`goGit()` keeps its open-and-cache behavior; its doc names `readGoGit` as its only caller and drops the "hold the lock for the whole use" instruction, which `readGoGit` now owns.

Route every listed method's go-git use through `readGoGit`, one call per read, and delete every direct `goGitMu` lock and `goGit()` call from them.
A wrapped read never calls another `Repo` method; a composite method such as `ChangedFilesSince`, `FileAtRevision`, `FilesInDirAtRevision` or `HeadContains` either does its whole go-git work in one `read` closure (tree resolution, diff and walk together) or calls `readGoGit` once per read in sequence.
`WorktreeChangedFiles` and `UntrackedFiles` run `repo.Worktree()`, the exclude read and `Status()` inside one `read`, under the shared lock D1 names in place of today's exclusive lock.
A method that legitimately expects a missing object maps `plumbing.ErrObjectNotFound` after `readGoGit` returns, so the retry happens before the "absent" answer: `SHAExists` folds it into false, and `HeadContains` into false with no error.
Error wrapping and every method's contract stay as they are.
Update `gogit.go`'s file comment to describe `readGoGit` in place of `lookupObjectRetrying`.

`internal/gitrepo/doc.go`'s packfile bullet describes the whole-operation retry: what `readGoGit` wraps, the snapshot comparison, the once-only retry, and the lock split.

Tests:

- `parity_test.go`: a new table-driven integration test is the #468 regression.
  Its fixture builds a `Repo`'s pack index (`forcePackIndexFreeze`), commits with the git CLI, runs `git repack -d` and `git prune-packed`, and each row asserts one method answers correctly afterwards: `UntrackedFiles` and `WorktreeChangedFiles` report the right paths, and `ChangedFilesSince`, `HeadContains`, `PathRevisions`, `CommitParents` and `SHAExists` each answer as for an unrepacked store.
  `TestSHAExists_MixedBackend_RepackBetweenCommitAndRead` becomes the table's `SHAExists` row and is deleted as a separate test.
- `gogit_test.go`: the `concurrent callers` subtest of `TestGoGit_StandaloneRepo` drives exported methods instead of the deleted `lookupObjectRetrying`; it gains a row in which two goroutines read one `Repo` over a stale pack index after a repack, both succeed, and the reindex counter rose by exactly one.
  A further subtest asserts a read for a genuinely absent sha, repeated with the pack set unchanged, reindexes at most once.
  Its file comment stops naming `lookupObjectRetrying`.
- `gogitcaller_test.go`: an untagged scan in package `gitrepo`, modeled on `noforceadd_test.go` with `scankit`, parses every non-test `.go` file of `internal/gitrepo` with `go/ast` and fails when any function other than `readGoGit` calls `goGit()`, with a vacuous-scan floor.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/gitrepo`'s row by the net count of new top-level tests (the regression table and the caller scan, less the absorbed `SHAExists` test).

PATTERN: add one entry under `## Fabric and git`, `PATTERN-gogit-read-helper` — reading through go-git in `internal/gitrepo` goes only through `readGoGit`, the sole caller of `goGit()`, marked `(test)`, with its detail in `pattern/PATTERN-gogit-read-helper.md`.
`PATTERN.md` is within a few hundred bytes of `MaxOverviewBytes`, and card 5 adds a second entry.
Keep both new entry lines short, carrying their detail in their background files, and free the remaining room only by removing clauses from existing entries that the entry's own background file already states, never by changing an invariant's meaning or raising the cap.
`TestOverview_RealFilePassesCheck` is the check.

**ImpactSummary:** Every go-git read in `internal/gitrepo` changes its locking and retry path behind unchanged method signatures; `lookupObjectRetrying` and its only test use go away.

**Commit:** 2: gitrepo: every go-git read retries whole after a repack
