---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/bolt.go:70
    summary: the unborn-HEAD branch and the nothing-stageable outcome of both commit routes have no test
    fix: In internal/fabricengine/bolt_integration_test.go add a row to TestBolt_CommitWritten whose write returns only a path neither on disk nor in HEAD, and assert committed is false, err is nil and HEAD is unchanged. In internal/fabricengine/boltpullfirst_integration_test.go add the same row to TestBolt_PullThenCommitWritten, exercising the `len(stageable) == 0` continue. Cover the unborn-HEAD branch (`head == ""`) with one row on a repo with no commits, in the existing CommitWritten test or a stageablePaths case in export_test.go form. Extend existing tests, add no new test function.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/fabricengine/bolt_integration_test.go:231
    summary: summary.md says the subtests assert the commit lands with exactly the other changes, but the CommitWritten subtest only checks that seed.md left git status
    fix: In the subtest "a deleted path is committed as a deletion and one HEAD never held is left out", assert `gitkit.Git(t, repoPath, "show", "--name-status", "--format=", "HEAD")` trimmed equals "D\tseed.md", as the pull-first subtest does for its commit.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/bolt.go:79
    summary: a missing path that HEAD holds as a directory is left out, because FileAtRevision reports ErrPathNotAtRevision for a non-file entry, while the doc comment says a missing path HEAD's tree holds is staged
    fix: In the stageablePaths doc comment state that paths are file paths and a directory is not recognised as held. No current caller passes a directory, so no code change is needed.
---

### [MEDIUM] Unborn-HEAD and nothing-stageable behaviors are uncovered

**Location:** internal/fabricengine/bolt.go:70
**Issue:** The diff adds the `head == ""` branch of `stageablePaths` and a nothing-stageable outcome in `CommitWritten` and `PullThenCommitWritten`.
No test pins any of them: the extended subtests always leave at least one stageable path, both fixtures seed a commit first, and the existing empty-list row returns nil, which hits the `StageAndCommit` guard rather than the new filter.

### [LOW] Summary overstates the CommitWritten assertion

**Location:** internal/fabricengine/bolt_integration_test.go:231
**Issue:** `_lyx/landing/summary.md` claims both subtests assert the commit lands with exactly the other changes.
The pull-first subtest does, through the name-status assertion on HEAD; the `CommitWritten` subtest checks only that `seed.md` is absent from `git status --porcelain` and that a commit exists.

### [LOW] Directory held by HEAD is treated as not held

**Location:** internal/fabricengine/bolt.go:79
**Issue:** `stageablePaths` tells held from not held through `FileAtRevision`, which returns `ErrPathNotAtRevision` for an entry that is not a file.
A missing path that HEAD holds as a directory is left out, where `git add` would have staged the removal.
Every current caller passes file paths, so nothing breaks today; the comment's "a missing one HEAD's tree holds" is accurate for files only.

## Rejected

- Lstat errors other than not-exist (such as `ENOTDIR`) send the path to `git add`, which refuses it: a narrow variant of the original failure, not reachable from the manifest names `renderOwnerWrite` returns, and the entry asks only for the stale-manifest case.
- The extra `CurrentSHA` read per write in `PullThenCommitWritten` is cheap and fails only on a corrupt repo.

## Checked and held

- `renderOwnerWrite` in `internal/boardengine/ownerset.go` is unchanged, matching the entry's expected result and the summary's stated design.
- The filter in `PullThenCommitWritten` runs after `pullFirst` and after each write, so the pull behaves as before; an error from the filter returns the commits already landed.
- Every caller of the two methods writes existing files or deletes tracked ones, so none changes behavior.
- Doc comments and the pull-first paragraph in `internal/fabricengine/doc.go` match the code, including the claim that a path staged in the index but absent from disk and HEAD is left out.
- No PATTERN rule is broken and no PATTERN change is needed.
