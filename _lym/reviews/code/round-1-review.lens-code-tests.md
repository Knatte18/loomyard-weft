```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that pins it, or none.

**Per behaviour, the test that pins it**

- **`showFile` reads the blob at HEAD, bytes unchanged (trailing whitespace kept).** Pinned by `TestShowFile_ReadsCommittedBytes/HeadKeepsTrailingWhitespace`.
- **`showFile` with a full commit SHA.** Pinned by `CommitSHA`. A regression that sent a full SHA through `ResolveSHA` would still pass, but the answer is the same, so nothing observable is lost.
- **`showFile` with a full ref name.** Pinned by `FullRefName`.
- **`showFile` with an empty file that is present.** Pinned by `NestedEmptyFileIsPresent`, which asserts a non-nil `[]byte{}`.
- **`showFile` for a path the revision lacks gives nil, nil.** Pinned by `AbsentPathIsNil` and `PathAddedLaterIsNilAtAnOlderCommit`.
- **`showFile` for an unresolvable rev returns an error.** Pinned by `UnknownRefIsAnError`.
- **`showFile` callers (`ReadConfigChanges`, `upstreamrun`).** Covered through the unchanged `TestReadConfigChanges` and the upstream-run tests; I did not open the latter.
- **`showFile` with a short SHA or an annotated tag, which `git show` accepted.** No row, so this is finding T1.
- **`showFile` for a path that is a directory.** The summary says the result changed from a tree listing to nil, and there is no row (finding T1).
- **The `sha256HexLength` branch.** No row, and a sha256 repository fixture is not worth adding. This is not a finding.
- **`RevParse("HEAD")` read from files in the main worktree on a packed ref.** Pinned by `TestRevParse_HeadReadFromFilesMatchesGit/PackedBranchInTheMainWorktree`.
- **`RevParse("HEAD")` from a subdirectory.** Pinned by `Subdirectory`.
- **`RevParse("HEAD")` in a linked worktree whose loose ref shadows a stale packed ref.** Pinned by `LooseRefShadowsPackedRef`.
- **`RevParse("HEAD")` on a detached HEAD.** Pinned by `DetachedHead`.
- **`RevParse("HEAD")` on an unborn branch falls back to git.** Pinned by `UnbornBranch`, which asserts `ok == false`.
- **`RevParse("HEAD")` in a bare repository falls back to git.** Pinned by `BareRepository`, which asserts `ok == false`.
- **`RevParse` with a non-HEAD rev goes to git.** The gitkit test has no row for it. It is exercised by many existing callers, such as `gitkit.RevParse(t, h.RecordsBare, "refs/heads/"+...)` in `internal/fabriccli`, which would fail if it broke. Adequately covered.
- **`headCommit` matches `dir` to its worktree by symlink-resolved path (`resolvedPath`).** No row, so this is finding T2.
- **`headCommit` returns false for a `dir` in no repository, or one that matches no listed worktree.** No row. A wrong answer here still falls back to git, so the cost is low (finding T2).
- **Leaf-import allowlist change.** Pinned by the existing `leaf_enforcement_test.go`, whose allowlist the diff updates. A test of the pattern text itself would be static prose, which is exempt.

**Findings**

- **T1 — LOW, class `scope`.** Location: `internal/fabricengine/configchanges_integration_test.go:25-65`, the `TestShowFile_ReadsCommittedBytes` table.
  - Summary: no row for a short SHA, an annotated tag, or a path that is a directory, though the summary documents the directory case as a deliberate change from git's tree listing.
  - Fix: add rows `ShortSHA` (`first[:7]`, want `first\n`), `AnnotatedTag` (tag created with `gitkit.Git ... tag -a`, want the commit's bytes) and `DirectoryPathIsNil` (rev HEAD, path `dir`, want nil).
  - Evidence: the table at lines 45-53 holds only HEAD, full SHA, `refs/heads/side` and path-absence rows, and the summary's "Design decisions" section names the directory change.
- **T2 — LOW, class `scope`.** Location: `internal/gitkit/gitkit_test.go:205-254`, the `TestRevParse_HeadReadFromFilesMatchesGit` table.
  - Summary: no row for a symlinked worktree path. That is the only reason `resolvedPath` exists, and no row would fail if it were removed.
  - Fix: add a `SymlinkedWorktree` row whose `dir` is `os.Symlink(repo, link)` with `fromFiles: true`, so the `ok` assertion fails if the symlink match breaks. Optionally add a non-repository `dir` row with `fromFiles: false`.
  - Evidence: the `headCommit` and `resolvedPath` code in `query.go` and the table rows at lines 235-242.

**Tool calls (0)**
I made no tool calls; my inherited context already held everything I needed.
