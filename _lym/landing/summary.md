# fabricengine and gitkit: read committed files and HEAD without git

Two frequent read-only git calls in `internal/fabricengine`'s tier 1 run are now answered in Go.
The package starts 2,601 fewer git processes per run, 515 of them in the serial phase.

## What changes

- `fabricengine`'s `showFile`, the read of a file's bytes at a revision, ran `git ls-tree --name-only <rev> -- <path>` and then `git show <rev>:<path>`.
  It now resolves rev through `gitrepo.ResolveSHA` when rev is not a full SHA, and reads the blob through `gitrepo.FileAtRevision`.
  Both are existing go-git reads.
  Every committed `.lym-anchor` read, the config-change reads, the records reads and the upstream run status read go through it.
- `gitkit.RevParse(tb, dir, "HEAD")`, which `CommitFile` also calls after every commit, reads HEAD through `dotgit.ListWorktrees` for the worktree holding dir.
  It asks git for any other rev, and also for a bare repository, refs that are not files, or a branch with no commit yet.
  `gitkit` may now import `internal/dotgit`.
  It already reached `dotgit` through `lymcwd`, so the import closes no cycle.
  `PATTERN-leaf-packages`, the `gitkit` package doc and its leaf allowlist test name it.
- `fabricengine`'s package doc says a committed file is read through `gitrepo`.

## Tests

- `TestShowFile_ReadsCommittedBytes` (fabricengine, tier 1) reads at HEAD, a commit SHA and a full ref name.
  It covers trailing whitespace kept, an empty file that is present, nil for an absent path, and an error for an unknown ref.
- `TestRevParse_HeadReadFromFilesMatchesGit` (gitkit, tier 1) checks that the files answer, equal to `git rev-parse HEAD`.
  It does so in the main worktree on a packed ref, a subdirectory, a linked worktree whose loose ref shadows a stale packed one, and a detached HEAD.
  For an unborn branch and a bare repository it checks that the answer is left to git.

## Measurement

Both runs are `lym gate test --wait --tier 1 ./internal/fabricengine -- -count=1 -exec "shard-runner.sh <dir> 1"`.
That is the one-process method of `docs/research/speed/fabricengine-shards-2026-11-09/report.md`, on the same machine.
The before run also served as the per-test count.
Each figure is one sample.

| Figure | Before | After |
|---|---|---|
| Git processes, whole package | 61,291 | 58,690 |
| Git processes in the 41 serial top-level tests | 11,666 | 11,151 |
| `git ls-tree --name-only` and `git show <rev>:<path>` | 962 | 92 |
| `git rev-parse HEAD` | 1,754 | 11 |
| Package time (`ok` line) | 35.6 s | 32.6 s |
| Phase 1, the serial chain | 20.4 s | 16.0 s |
| One-minute load before the run | 17.9 | 5.7 |

The walls are not a clean comparison: the before run started at three times the after run's load.
The process counts do not depend on load.
The serial tests are the same 41 in both runs.

### Where the serial phase's git processes came from

The serial tests' processes were attributed by the test's temp directory in each process's trace2 `def_repo` event.

- Kept, because they write: the push and fetch pipelines (`receive-pack`, `upload-pack`, `pack-objects`, `unpack-objects`, `rev-list --objects`, `for-each-ref` under `receive-pack`), commits, hub-copy amends, `update-ref`, `worktree repair`.
  Also kept: `housekeeping.go`'s `git config gc.auto 0` and `git config maintenance.auto false`, 264 each in the serial phase and 1,122 each in the package.
- Kept, outside the two packages the task may change: `gitrepo`'s `rev-list --count @{u}..HEAD` (238), `remote get-url --push` (148) and `rev-parse --verify --quiet MERGE_HEAD`.
  So are the post-checkout hook's shell `rev-parse` calls.
- Kept, because Go cannot answer the same way here:
  - `rev-parse --git-path hooks` (111) and `git remote` (102) read every config scope, includes and `core.hooksPath` among them.
  - `branch --format=%(refname:short)…%(worktreepath)` (190) shortens a name differently when a tag or remote shares it, and adds a line for a detached HEAD.
  - `ls-files -- <path>` (170) would need an index read by path that `gitrepo` does not offer.
  - `status --porcelain` (285) has no exact Go equivalent.
  - `rev-parse --abbrev-ref HEAD` (85) shortens a name the way `git branch` does, so Go cannot match it either.
- Replaced: `showFile`'s `ls-tree` and `show` pair (230), and `gitkit.RevParse`'s `rev-parse HEAD` (223).

## Design decisions

- `showFile` passes a full SHA straight to `FileAtRevision` and resolves any other rev with `ResolveSHA`.
  A full SHA therefore never goes through ref lookup, as with git.
- One answer differs from git, in a case no caller reaches.
  For a path that is a directory at rev, `git show` printed the tree listing, and `showFile` now reports the path absent.
  Every caller asks for a file.
- `gitkit.RevParse` matches dir to its worktree by its symlink-resolved root.
  Where the files cannot answer for certain, it asks git rather than guess, so its result and its failures stay git's.

## Left for later

- `disableStoreAutoGC` writes the two gc keys on every removal: 2,244 processes in the package, 528 in the serial phase.
  When the shared config already holds both values, the write changes nothing.
  The task keeps every write, so this stays as it is.
- An index read by path in `gitrepo` would let `PathTracked` stop running `ls-files`.
- `gitrepo`'s `MERGE_HEAD` probe and `rev-list --count @{u}..HEAD` are the largest read-only calls outside this task's packages.
