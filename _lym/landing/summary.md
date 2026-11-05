# tmuxkit: fail a package that leaves a .git in a shared temp directory

An empty `/tmp/.git` once appeared during a tier1 run.
With it in place, go's VCS stamping and git's repository discovery treated everything below `/tmp` as one repository, and `TestPrebuildLym` failed.
No test or process was found to have created it, so this change adds a guard that names the package next time instead of hunting the writer now.

## What changes

`tmuxkit.Main` snapshots whether a `.git` exists directly in two shared directories before `m.Run()`: the lym temp root and the system temp directory above it.
After the run it fails the package with exit 1 for each `.git` that is new, naming the path.
An empty directory is removed so it breaks no later build; a non-empty directory, a gitfile or a symlink is kept and reported for removal by hand.
A symlink is never followed.
The check runs on Windows as well as on Linux and macOS.

`pattern/PATTERN-test-isolation.md` documents the check and its bound.

## Design decisions

- The guard watches only a new `.git`, not any new entry under `/tmp`, which departs from the board entry's idea.
  `/tmp` and the lym temp root gain legitimate entries from concurrent binaries and other programs, so a check on any entry would fail innocent packages.
  A `.git` there is never legitimate.
- The leaker is not found in this task.
  A search of every `.git`-creating, `TempDir`-reading, chdir and subprocess-environment site found no path to `/tmp`.
  The guard attributes the next leak to its package.
- Bound: the check cannot tell which binary wrote the entry, so a package running at the same time can be failed for it.
  A `.git` present before the run is no finding, and a binary killed by a panic or timeout skips the check.

## To check by hand

- `Main`'s wiring of `runCheckingGitEntries` is covered only through the helper's tests, not by a test that runs `Main` itself; read the two call sites in `internal/testkit/tmuxkit/tmuxkit.go` (the Windows branch and the Unix branch).
- The leaker of the original `/tmp/.git` is still unknown.
  When a package fails with this finding, look for a path joined onto an empty or parent directory in that package's tests.
