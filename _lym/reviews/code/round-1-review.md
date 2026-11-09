---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/configchanges_integration_test.go:45
    summary: TestShowFile_ReadsCommittedBytes has no row for a short SHA, an annotated tag or a directory path, though the summary records the directory answer as changed from git's tree listing to nil
    fix: In the TestShowFile_ReadsCommittedBytes table add a ShortSHA row (rev first[:7], path a.txt, want "first\n"), an AnnotatedTag row (create the tag with gitkit.Git(t, repo, "tag", "-a", "-m", "tag", "v1", first) before the table, rev "v1", want "first\n") and a DirectoryPathIsNil row (rev HEAD, path "dir", want nil).
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/gitkit/gitkit_test.go:235
    summary: TestRevParse_HeadReadFromFilesMatchesGit has no row for a symlinked worktree path, the only case resolvedPath exists for, so removing it would fail no test
    fix: In the TestRevParse_HeadReadFromFilesMatchesGit table add a SymlinkedWorktree row. Create the link with os.Symlink(repo, filepath.Join(t.TempDir(), "link")) before the table and use it as dir with fromFiles true.
---

Range reviewed: `git diff bf05b2ef..HEAD`, three commits, nine files, all inside `internal/fabricengine`, `internal/gitkit` and `pattern/`.
The scope matches the decision record's limits, and no commit carries a `Pattern-Departure:` trailer, so none is needed.

Holistic checks that raised nothing:

- The decision record asks for a count, a replacement and a before/after measurement in the landing note.
  `_lym/landing/summary.md` records all three, and the process-count rows match what the diff replaces (`showFile`'s `ls-tree` and `show` pair, and `rev-parse HEAD` in `gitkit.RevParse`).
  I did not rerun the measurement.
- The declared PATTERN change lands:
  `pattern/PATTERN-leaf-packages.md`, the `gitkit` package doc and `leaf_enforcement_test.go` all list `dotgit`.
  `dotgit` imports only the standard library, so `gitkit` reaches no feature package through it.
- `gitexec.Run` returns stdout unmodified, so `showFile` returning the blob's bytes unchanged keeps what the old `git show` path returned.
- Each of the four `showFile` callers passes a branch tip, a SHA or a tracking ref.
  `ResolveSHA` handles the non-SHA ones, and no caller type-asserts a `*gitexec.GitError` from `showFile`.
- Both Go reads fall back to git, or err where git erred, in the cases the focus file names: an unborn branch, a bare repository, non-`files` ref storage, a missing path and an unresolvable rev.
- Two differences from git, both in cases no caller reaches: a directory path now reads as absent (the summary states it), and a submodule gitlink path reads as absent where `git show` errored (the summary does not mention it).
  Neither changes what a production path decides, so I raise no finding.

## Rejected

- Submodule gitlink path in `showFile` reads as absent where `git show` errored (lens:code-correctness): no caller asks for a gitlink path, so no production decision changes.
- `headCommit` returns a `Head` that is not a full SHA unvalidated (lens:code-correctness): reachable only through a corrupt repository, where the old `git rev-parse` would also have failed in the same environment.
- No row for the `sha256HexLength` branch (lens:code-tests): a sha256 fixture repository costs more than the one-line length test it would pin.

### [LOW] No test row for short SHA, annotated tag or directory path in showFile

**Location:** internal/fabricengine/configchanges_integration_test.go:45
**Issue:** The table pins HEAD, a full SHA, a full ref name, an empty file and absent paths.
A short SHA and an annotated tag still go through `ResolveSHA`, which git's `rev:path` also accepted, and neither has a row.
The directory-path answer changed from git's tree listing to nil, and the summary states that, yet no row pins it.

### [LOW] No test row for a symlinked worktree path in headCommit

**Location:** internal/gitkit/gitkit_test.go:235
**Issue:** `resolvedPath` exists so that a `dir` reached through a symlink still matches its worktree.
The table has no symlinked `dir`, so deleting `resolvedPath` would leave every row green while the fast path silently fell back to git.
