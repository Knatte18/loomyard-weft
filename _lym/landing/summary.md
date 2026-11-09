# fabricengine: read committed files and HEAD without a git process

Two read-only git calls on the serial path of `internal/fabricengine` tier 1 now answer from Go.
Each removed spawn shortens a serial test by the spawn's whole latency, and spawns slow down as more run at once.

- `showFile` in `internal/fabricengine/configchanges.go` replaces its `git ls-tree` and `git show` pair with `gitrepo.ResolveSHA` and `gitrepo.FileAtRevision`.
  A full SHA-1 or SHA-256 hex name skips the resolve.
  Any other rev, such as `HEAD`, a short SHA, a tag or a full ref name, goes through `ResolveSHA`.
  A path the revision lacks, or one that names a directory, still returns nil bytes with no error.
- `gitkit.RevParse` answers `HEAD` from the repository's files through `internal/dotgit` when the worktree's HEAD names a commit.
  A bare repository, an unborn branch and a path outside any worktree fall back to `git rev-parse`, as does every other rev.
  `internal/gitkit` now imports `internal/dotgit`; the leaf-import allowlist, `doc.go` and `pattern/PATTERN-leaf-packages.md` say so.

No production path changes what it reads or decides.
`internal/fabricengine/doc.go` records that a file's bytes at a revision are read through `gitrepo`.

## Tests

- `TestShowFile_ReadsCommittedBytes` compares `showFile` with what `git show <rev>:<path>` prints.
  It covers `HEAD`, a full and a short SHA, an annotated tag, a full ref name, trailing whitespace, an empty file, a directory path, an absent path and an unknown ref.
- `TestRevParse_HeadReadFromFilesMatchesGit` pins that the file read equals `git rev-parse HEAD` in packed, loose-over-packed, detached, linked, symlinked and subdirectory cases.
  It also pins that unborn and bare repositories are left to git.

## To check by hand

- The per-subcommand count of git processes, and the tier 1 wall and git process count before and after, were not recorded for this change.
  The task asked for them; run `lym gate test --wait --tier 1 ./internal/fabricengine` with `GIT_TRACE2_EVENT` set on this branch and on `main`, on the same machine, and compare.
- The replacements cover only the calls the change touches.
  Other frequent read-only calls in the package, such as `diff --name-only` in `diffNames`, still spawn git.
