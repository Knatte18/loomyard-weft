# Card 23 — dotgit-leaf

**Create:**
- `plan:internal/dotgit#Geometry` -> `type Geometry struct`
- `plan:internal/dotgit#ErrNotRepository` -> `var ErrNotRepository = errors.New("dotgit: not a git repository")`
- `plan:internal/dotgit#Read` -> `func Read(dir string) (Geometry, error)`
- `plan:internal/dotgit#IsRepository` -> `func IsRepository(g Geometry) bool`
- `plan:internal/dotgit#FindRoot` -> `func FindRoot(start string) (root string, g Geometry, err error)`
- `plan:internal/dotgit#readGitFile` -> `func readGitFile(path string) (string, error)`
- `plan:internal/dotgit#readCommonDir` -> `func readCommonDir(gitDir string) (string, error)`
- `internal/dotgit/doc.go`
- `internal/dotgit/dotgit_test.go`
- `internal/dotgit/leaf_enforcement_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `docs/shared-libs/README.md`

**Intent:** D12's new leaf: the one `.git` parser, importing the standard library alone.
`Geometry` carries `GitDir` and `CommonDir`, both absolute.
`Read` reads `dir`'s `.git` entry: a directory is the git dir; a file is a gitfile whose `gitdir:` line, relative to `dir` when not absolute, is the git dir; the git dir's `commondir` file, relative to the git dir when not absolute, is the common dir, and the git dir itself otherwise.
A missing `.git` entry, a gitfile without a `gitdir:` line, or a `gitdir:` or `commondir` target that does not exist is `ErrNotRepository`, wrapped with the path, so the caller can tell a pruned worktree from a plain directory by the message.
`IsRepository` is the repository test of the amended D12: `HEAD` present in the git dir, and `objects` and `refs` directories present in the common dir.
`FindRoot` walks up from `start`, as `rev-parse --show-toplevel` does: a directory whose `.git` is a directory ends the walk only when `IsRepository` holds, and is skipped otherwise, as git skips it; a directory whose `.git` is a file ends the walk, and a target that is missing or fails `IsRepository` is `ErrNotRepository`, never resolved to an enclosing repository; reaching the filesystem root is `ErrNotRepository`.
It lives in `internal/dotgit/dotgit.go`; `doc.go` explains why it is its own package: `gitrepo`'s in-package test imports `gitkit`, which reaches `lyxcwd` through `configengine` and `logger`, so `lyxcwd` importing `gitrepo` would close a cycle in that test.
`dotgit_test.go` is untagged and builds every layout by hand with `os.MkdirAll` and `os.WriteFile`: a `.git` directory with and without `HEAD`, `objects` and `refs`; a gitfile with an absolute and a relative target; a `commondir` file present and missing its target; an empty `.git` directory inside a repository, which resolves outward; a gitfile with a missing target inside a repository, which is refused; a directory under no repository.
`leaf_enforcement_test.go` pins the import set to the standard library through `scankit.AssertImportAllowlist`.
The budget gains the row `internal/dotgit: 3`.
`PATTERN.md`'s `PATTERN-leaf-packages` entry names `dotgit`, and the background file gains an `internal/dotgit` section stating the standard-library-only cap and that it is the sole `.git` parser; `docs/overview.md`'s module tree gains the `internal/dotgit/` row beside `internal/lyxcwd/` and the shared-infrastructure list names it; `docs/shared-libs/README.md`'s implementation-only list gains it.

**ImpactSummary:** A new package with no callers yet; one budget row and four doc files.
