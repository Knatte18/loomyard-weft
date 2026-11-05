# Fail a test package that leaves a .git in a shared temp directory

An empty `/tmp/.git` appeared during a tier1 gate run.
With it in place, go's VCS stamping ran git in `/tmp` and failed `TestPrebuildLym` on another branch.
This change makes the test harness catch such a leak at the package that causes it, and clean up after it.

## What changes

`tmuxkit.Main`, which every test package's `TestMain` runs through, now checks the directories above its run directory that every test binary on the machine shares: the lym temp root (`/tmp/lym-<uid>`) and the system temp directory that holds it.
Before `m.Run` it records which of them already hold a `.git`.
After the run it fails the package with exit 1 for each `.git` that appeared in the meantime, and names the path on stderr.
It removes the entry when it is an empty directory, so the stray breaks no later build.
Any other entry, such as a populated repository or a gitfile, is left in place, and the message asks for it to be removed by hand.

`PATTERN-test-isolation.md` and the tmuxkit package doc describe the new check and its bounds.

## Design decisions

- **The check is narrowed to `.git`.**
  The board entry asked for a guard against any new entry directly under `/tmp` or the gate's temp root.
  `/tmp` is shared with every program on the machine, and the lym root legitimately gains run directories and creel sockets from concurrent binaries, so a check on any new entry would fail innocent packages all the time.
  A `.git` in either directory is never legitimate, and it is the entry that breaks VCS stamping and git discovery for everything below it.
- **The guard is in testkit, not in the gate.**
  A gate-level check would see every package of a run at once, and of every concurrent gate on the machine, so it could not tell which run wrote the entry.
  `tmuxkit.Main` attributes it to the test binaries that were running when it appeared.
  Bound: a package running at the same time also fails for the same entry. An entry present before the run is no finding.

## Leaker not found

The board entry also asked to find the code that creates the `.git`.
A search of every site that creates a `.git` entry, reads `os.TempDir()`, changes the working directory or builds a subprocess environment found no path that reaches `/tmp` itself.
Every fixture writes under its own `t.TempDir()`, and `tmuxkit.Main` points `TMPDIR` at the run directory before any test runs.
The weakest remaining candidates are the bare-remote fixture helpers that join `name+".git"` onto a directory, in `internal/fabriccli/cli_test.go` and `internal/fabricengine/clone_adopt_test.go`, and an interrupted `git init` or `git clone`.
None has a caller that passes `/tmp`.
The new check names the failing package the next time the leak happens, which narrows the search to that package's tests.

## Tests

`TestCheckNewGitEntries` in `internal/testkit/tmuxkit` covers these cases:
- no entry;
- an entry present before the run, which is kept with the code unchanged;
- a new empty directory, which fails and is removed;
- a new non-empty directory, which fails and is kept;
- a new gitfile, which fails and is kept.

`TestSharedTempDirs_AreTheRootAndTheSystemTempDirectoryAboveIt` pins which two directories the check watches.
