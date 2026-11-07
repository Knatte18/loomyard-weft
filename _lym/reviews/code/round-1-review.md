---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dotgit/worktrees_test.go:56
    summary: ListWorktrees branches the diff adds have no row - explicit refStorage files, symref chain and depth limit, ENOTDIR loose ref, non-directory worktrees entry, symlinked or missing relative gitdir target
    fix: Add rows to the TestListWorktrees table in internal/dotgit/worktrees_test.go - a config with extensions.refStorage = files that lists normally; a HEAD naming a ref whose file holds another ref line, wanting the last ref of the chain as Branch; a chain of six links wanting an error; refs/heads/a/b with refs/heads/a a file, resolving through packed-refs; a plain file under worktrees/ that is skipped; a relative gitdir whose admin dir is reached through a symlink or whose target does not exist. Do not add a new test function.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitkit/query.go:125
    summary: SkipBelowGitVersion and compareDottedVersions parse git version strings with no test, so a misparse skips the version-gated tests silently
    fix: Add one table-driven test beside the gitkit tests that feeds version strings such as 2.53.0, 2.47.1.windows.1 and a 2.39.5 Apple Git line to the comparison, through an export_test.go hook or by extracting the version-string parse into a function the test can call.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/commit_integration_test.go:287
    summary: comment says a missing anchor subdirectory makes git worktree list fail, but List now fails inside dotgit before any git process
    fix: In internal/fabricengine/commit_integration_test.go:287 reword the comment to say the absent anchor subdirectory makes the worktree list read fail in the anchor path.
---

### [LOW] Unpinned ListWorktrees branches

**Location:** `internal/dotgit/worktrees.go:121`, `:149-183`, `:247-295`, `:339-380`; table at `internal/dotgit/worktrees_test.go:56`.
**Issue:** Every acceptance criterion has a test, including parity with git, the symlinked path, the removed gitdir and HEAD, the packed-only ref, the reftable fallback for both callers, no git process, the subdirectory and the missing directory.
The branches the diff adds beyond those have no row that would fail if they broke.
The change description claims the five-link symbolic-ref chain and the last-ref-of-chain branch rule, yet no row nests a symbolic ref.
`extensions.refStorage = files` is never listed, so a reader that refused every explicit value would pass.
The `ENOTDIR` and `EISDIR` not-found path, the skip of a non-directory entry under `worktrees/` and the symlink-resolving branch of `readAdminGitdir` have no row either.
The relative-gitdir tests resolve their root through `EvalSymlinks` first and always have an existing target, so `resolveExistingPrefix` never reaches its walk-up.

### [LOW] Version-skip helper untested

**Location:** `internal/gitkit/query.go:125-157`.
**Issue:** `SkipBelowGitVersion` gates the relative-gitdir parity test and the reftable subtests.
A parse that mishandles a suffix such as `2.47.1.windows.1` or `(Apple Git-154)` makes those tests skip with a wrong version, and the acceptance criteria they pin go unchecked with no failure.
The helper sits in `gitkit`, whose closed import set excludes `toolfloor`, so a second parser there is licensed.

### [NIT] Stale comment on where the failure arises

**Location:** `internal/fabricengine/commit_integration_test.go:287`.
**Issue:** The comment says the absent anchor subdirectory makes `git worktree list` fail.
`List` now rejects the directory in `dotgit`'s `requireDir` before any git process, and the test still passes through the wrapped error.

## Rejected

- Lens code-affected-set F1 (second version parser beside `toolfloor`): `toolfloor` exports no version compare, only `CheckGit`, `CheckTmux` and `TmuxHasAllowSetTitle`, and `gitkit`'s closed import set in `PATTERN-leaf-packages` excludes it.
- Lens code-affected-set F2, the `reconcile.go:188`, `hubmutation_integration_test.go:346` and `destructivegaps_integration_test.go:788` comments: the `reconcile.go` comment says only "The worktree list is read before the loop", which stays true, and the other two are loose wording that no longer misleads; only the `commit_integration_test.go` comment misplaces the failure, and it is kept as F3.
- Lens code-correctness F1 (one unreadable or over-deep linked entry fails the whole list): the decision record asks only that a missing `HEAD` never fails, and an unreadable file or a runaway symbolic ref is an operational error that the code returns with its path, as `code-quality` asks.
- Lens code-correctness F2 (empty or garbage `HEAD` or loose ref gives an empty `Head`): git writes neither state, the record does not ask for it, and the repository's own files are trusted input.
- Lens code-tests F3, F5 and F6 as separate findings: folded into F1, since each is a row in the one existing table.
- Lens code-tests observation that the `(detached)` mapping has no new assertion: the existing `worktreelist_test.go` tests cover the mapping, and a new row would duplicate them.

## Holistic check against the decision record

Each D1 bullet and acceptance criterion is made by the diff:

- Order, symlink resolution, the relative gitdir, the omitted entries, prunable, the missing HEAD, the detached and unborn values and the bare error all match the amended entry.
- The reftable fallback in both callers triggers on `ErrRefStorage` alone through `errors.Is`.
- A missing directory fails in `requireDir` with no walk-up, so the `junction.go` and `scratchlink.go` fallbacks still fire.
- `dotgit` stays standard-library only and spawns nothing.
- The `PATTERN-leaf-packages` background lists the worktree entries, refs and config keys, and the declared change landed.
- Neither commit carries a `Pattern-Departure:` trailer, and none is needed.
- The `gitkit/query.go` helper is the shared version skip the acceptance criteria require, so it is inside the scope.
- The change description's claims hold against the diff.
