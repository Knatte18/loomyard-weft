```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the pinning test**
- Main worktree first, with its path resolved through symlinks: `TestListWorktrees` row "a symlinked path…", and the `link` start in `TestListWorktrees_MatchesGitPorcelain`. Pinned.
- Linked worktrees sorted by path: `TestListWorktrees` row "linked worktrees follow sorted…". Its ids z, a, m, p are the reverse of the path order, so id order cannot pass by accident. Pinned.
- `core.ignorecase` fold and bytewise sort: the two "linked paths sort…" rows. The `; folded` inline comment is covered in the same row. Pinned.
- Relative `gitdir`: the hand-laid row and `TestListWorktrees_RelativeGitdirMatchesGit`. The latter skips below 2.48 and names the version. Pinned. The symlink-resolving branch of `resolveExistingPrefix` and `readAdminGitdir` has no row, because the test root is already `EvalSymlinks`-resolved (F5).
- `gitdir` missing or empty gives an omitted entry: the "gitdir file is missing or empty" row, plus `no-gitdir` in the git test. Pinned. The unreadable case shares the same branch.
- Prunable only when unlocked and `.git` missing: `d-gone` is prunable and the `a-locked` row (`created=false`, lock file present) is not. Pinned.
- Locked with and without a reason: `locked-reason` and `locked` in the git test. Pinned.
- No `HEAD` file gives detached at the all-zero SHA: the "without a HEAD file" row, plus `no-head` in the git test. Pinned.
- Packed-only ref, unborn branch, sha256 zero SHA, a peeled `^` line in packed-refs: rows 3, 4 and 5. Pinned.
- Detached SHA in `HEAD`: row 2 `c-detached`. Pinned.
- Bare repository stays an error: the "bare repository is refused" row. Pinned.
- Missing directory gives an error with no walk-up, and a directory in no repository: those two rows, and `TestList_Errors`. Pinned.
- Subdirectory of a linked worktree finds its repository: a row, the git test, and `TestList_RefStorage`. Pinned.
- `ErrRefStorage` for reftable: the dotgit row, and the reftable subtests in `TestList_RefStorage` and `TestOtherWorktrees`. These check the fallback result for both callers. Pinned. An explicit `refStorage = files` is never exercised (F2).
- `fabricengine.List` and `otherWorktrees` start no git process: the `FilesBackendStartsNoGit` subtests, which empty `PATH`. Pinned.
- `List` mapping to `WorktreeEntry` (short branch name, `Main` on the first entry, `Prunable`): `TestList_RefStorage` and the existing `TestList_ParsesPrunable`. The `(detached)` mapping is not asserted by any new test, but the existing `worktreelist_test.go` tests probably cover it.
- Fallback only on `ErrRefStorage`: indirectly pinned by the missing-directory row of `TestList_Errors`, which asserts `errors.Is(fs.ErrNotExist)`. A fallback to git would not wrap that error.
- `SkipBelowGitVersion` and `compareDottedVersions`: none (F4).
- Symbolic-ref chain (`resolve`, `maxSymrefDepth`, branch reported as the last ref of the chain): none (F1).
- Loose-ref `ENOTDIR` and `EISDIR` treated as not found, and unreadable-file or malformed `packed-refs` error paths: none (F3).
- Config parser details (`[core] bare = true` on one line, subsection headers, last assignment wins, case-insensitive keys): none (F2).
- A non-directory entry under `worktrees/` is skipped: none (F6).

**Findings**
- **F1** — LOW, design. `internal/dotgit/worktrees.go:247-270`, `resolve` and `maxSymrefDepth`. No row has `HEAD` pointing at a ref that is itself symbolic, such as `refs/heads/a` containing `ref: refs/heads/b`, and none nests past five levels. The "last ref of the chain" branch rule and the depth error are unpinned. The fix is two rows in `worktrees_test.go`: one for a two-link chain and one for six links.
- **F2** — LOW, design. `internal/dotgit/worktrees.go:339-380`, `readRepositoryConfig`. Only `reftable`, `ignorecase`, `bare` in its own section, and `objectFormat` are exercised. An explicit `refStorage = files` is never tested. A bug that refused every explicit value, as opposed to `files`, would pass. Neither the "last assignment wins" rule, nor the case-insensitive key names, nor a header with trailing content on the same line (`[core] bare = true`) has a row. Add rows to the existing table; the `files` row is the most important.
- **F3** — LOW, design. `internal/dotgit/worktrees.go:279` and `:306`. The `ENOTDIR` and `EISDIR` loose-ref cases have no row. A ref such as `refs/heads/a/b` whose parent `refs/heads/a` is a file falls through to `packed-refs`. The unreadable-file and `packed-refs` read-error paths are untested too. One row for the not-found case is enough; the others are operational paths.
- **F4** — LOW, design. `internal/gitkit/query.go:125-156`, `SkipBelowGitVersion` and `compareDottedVersions`. The new parser handles suffixes such as `2.43.0.windows.1` or `(Apple Git-154)`, and it has no test. A wrong parse makes the version-gated tests skip silently, so the acceptance criterion for the relative `gitdir` could go unchecked. Add one table-driven test of `compareDottedVersions`, or a version-string extractor that a test can feed.
- **F5** — LOW, design. `internal/dotgit/worktrees.go:149-183`, `readAdminGitdir` and `resolveExistingPrefix`. Both tests resolve the root through symlinks first, so the symlink-resolving branch is never reached. The same applies to the path when the relative target does not exist yet. Add a row for a relative `gitdir` whose admin directory is reached through a symlink, or a relative target that does not exist. Alternatively, drop the resolution if git's behaviour does not need it.
- **F6** — NIT, design. `internal/dotgit/worktrees.go:121`. The skip of a non-directory entry under `worktrees/` has no row. It is a one-line addition to an existing row.

**Test-economy observations (not findings)**
- I found no duplicated coverage. The `dotgit` tests, `TestList_RefStorage` and `TestOtherWorktrees` each pin a different layer.
- `TestList_RefStorage` and `TestOtherWorktrees` are not parallel, and each says why in its comment.

Tool calls: none; my inherited context held everything this lens needed.
