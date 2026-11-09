```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens on the `showFile` and `RevParse`/`headCommit` changes. Each entry gives the case I ran and whether it was correct.

**`showFile` (`internal/fabricengine/configchanges.go`)**
- Full 40-hex SHA: skips `ResolveSHA` and goes to `FileAtRevision`. Correct.
- Full 64-hex SHA: same path, and `validSHA` accepts 4 to 64 hex characters. Correct.
- `HEAD`, which is not hex: `ResolveSHA` returns the commit. Correct. On an unborn HEAD it returns an error, and `ls-tree` errored there too. Correct.
- Odd-length hex such as `abc`: `DecodeString` fails, so it is resolved. Correct.
- Even-length short hex such as `abcd`: decodes, but the length is wrong, so it is resolved by prefix. Correct.
- Uppercase 40-hex SHA: decodes and is accepted by `validSHA`. Correct.
- Annotated tag name: `ResolveRevision` peels to the commit (go-git `repository.go:1612-1621`). Correct, and git's `rev:path` also peels.
- Full ref name `refs/heads/side`: resolved. Correct, and the test pins it.
- Unknown ref: `ResolveSHA` returns a wrapped error, as git did. Correct.
- Path absent at rev: `ErrPathNotAtRevision` becomes `nil, nil`. Correct.
- Empty file: returns `[]byte("")`, which is non-nil, so a present empty file stays distinct from absent. Correct.
- Trailing whitespace: bytes pass through unchanged. Correct.
- Path is a directory: `tree.File` returns `ErrFileNotFound`, so the result is `nil, nil`. Git printed a listing. This is a known and documented difference, and no caller asks for a directory. Not a defect.
- Path is a submodule gitlink: it is not a file mode, so the result is `nil, nil`. `git show` errored here, and `ls-tree` was non-empty. This is a difference from git that the summary does not mention. No caller reaches it. LOW at most, and I am not raising it.
- Symlink path: `Mode.IsFile` is true for a symlink, so the target text is returned, as `git show` does. Correct.
- Rev is a full 40-hex string that is also a branch name: git prefers the object name, and so does this code. Correct.
- `dir` not a worktree root: `gitrepo` bans `DetectDotGit` (`doc.go:261`), whereas git would walk up. `ReadConfigChanges` and `upstreamrun.go:72` pass `weftRepoRoot` and `weftPath`, which look like roots. I did not trace every caller of `ReadConfigChanges`, so I am not raising it.
- Variable scoping: the inner `err` in `if sha, err = repo.ResolveSHA(rev)` assigns the outer `sha` and does not shadow it. Correct.

**`RevParse` and `headCommit` (`internal/gitkit/query.go`)**
- Rev other than `HEAD`: falls through to git. Correct.
- Main worktree on a packed ref: `ListWorktrees` reads packed-refs. Correct, and the test pins it.
- Loose ref over a stale packed one: `resolve` tries the loose file first. Correct, and the test pins it.
- Subdirectory of a worktree: `FindRoot` walks up. Correct.
- Linked worktree: matched by the admin `gitdir`, stripped of `.git`. Correct.
- Symlinked path: both sides go through `EvalSymlinks`. Correct. The unresolved `FindRoot` root is why `resolvedPath` is needed.
- Detached HEAD: the raw HEAD content is the SHA. Correct.
- Unborn branch: `Head` is all zeros, so it falls back to git, which fails or prints as before. Correct.
- Empty HEAD ref content: the trim leaves `""`, so it falls back. Correct.
- Bare repository: `FindRoot` fails, so it falls back. Correct.
- Reftable (non-`files` ref storage): `ErrRefStorage` makes `ListWorktrees` return an error, so it falls back. Correct.
- Nonexistent dir: `requireDir` fails, so it falls back, and git then fails loudly as before. Correct.
- Submodule worktree (`.git/modules/x`): the main-worktree path comes from `stripDotGit(commonDir)`, which never matches `root`, so it falls back to git. Safe.
- `core.worktree` set: it likely fails to match and falls back. Safe.
- HEAD naming a ref that is a symref chain deeper than 5: `resolve` errors, so it falls back. Correct.
- Concurrent ref update: git replaces the file atomically, so no torn read. Correct.
- Worktree `Head` that is not a SHA, for example a corrupt ref: the code returns it unvalidated, where git would error. The only ways to reach this are a corrupt repository or a HEAD file with odd content. LOW at most, not worth a finding.

**Findings:** none at MEDIUM or above for this lens. Both reads match git's answer or fall back to git for every case I ran. The one unreported difference is the submodule-path case in `showFile`, which is LOW and unreachable by any caller.

**Tool calls**
1. Grep for `treeForRev`, `PlainOpen`/`DetectDotGit` and `validSHA`: the manifest lacked the open options and the commit-only requirement of `treeForRev`.
2. Grep of go-git `ResolveRevision` for tag peeling: the manifest lacked its annotated-tag behaviour.
3. Read of go-git `ResolveRevision` plus a grep for `shaPattern`: to confirm the peel-to-commit logic and the 4 to 64 hex bound.
