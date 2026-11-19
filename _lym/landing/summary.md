# tmuxkit: fail a stray temp-dir `.git` only on a package that ran alone

`tmuxkit.Main` used to fail every test package that was running when a `.git` appeared directly in `/tmp` or the lym temp root.
It could not tell which binary wrote the entry, so it failed every package that ran at the same time as the writer.
The `_bolt` gate logs hold 22 such runs between 2026-11-06 and 2026-11-18.
Every one removed an empty `/tmp/.git`, and none failed the package that wrote it.
The writer's own run always passed, because a package that finished earlier had already removed the entry.

## What changes

- `Main` watches both shared directories through `internal/fswatch` while `m.Run` runs.
- On Linux, the moment a `.git` is created there, it reads `/proc` for two things:
  - every other `*.test` binary running, leaving out this binary's own descendants;
  - every process whose working directory is that directory.
- The package fails with exit 1 only when no other test binary was running at that moment.
- Otherwise `Main` prints a machine finding with the time of the sighting, the binaries running then and the processes in the directory, and it keeps the exit code.
  Across occurrences, the binary that every finding names is the writer.
- An entry the watch saw is reported even when another package already removed it.
  So every package running at that moment adds its own view to the gate log.
- An empty `.git` is still removed, and anything else is still named for removal by hand.
- The check moves from `tmuxkit.go` into its own file, `internal/testkit/tmuxkit/gitentry.go`.
  The package doc and `pattern/PATTERN-test-isolation.md` describe the new rule and its limits.

## Design decisions

- **The writer is not fixed in this change.**
  The board entry asked for the writer to be found and fixed.
  - Timing narrows it down: `loomcli` and `reedcli` are the only packages that were running in or beside all 22 failing runs.
    In the 2026-11-08 tier2 run, only `pairteardown`, `reedcli` and `loomcli` were still running when the entry appeared.
    `webstercli` and `creelcli` had already finished.
    For comparison, three random sets of 22 gate runs had no package in common.
  - Reading both packages and the code their tests drive found no path that reaches `/tmp`.
    This matches the search behind dfe40fd5.
  - Their tier0 and tier1 gates passed with no finding while a watcher polled `/tmp/.git`.
    So the write needs something those runs did not have, perhaps another package running at the same time.
  - The new finding records the binaries running at the moment of the write, which is the information this hunt lacked.
    Without it, the hunt could only match whole gate logs against each other.
- **Blame needs the package to have run alone.**
  inotify does not report which process made a change, and fanotify needs privileges that test binaries lack.
  So when the entry appeared, a package can only know which other binaries were alive, not which one wrote it.
  Failing only a package that ran alone never fails an innocent package that ran beside another test binary.
- **No failure off Linux, or without a sighting.**
  With no process table to read, the run cannot be blamed, so the finding there is reported and never fails the package.
  This also covers an entry created between the start-of-run snapshot and the moment the watch opens.
- **Known gap:** a writer that is not a test binary, such as a daemon left by a finished test, is blamed on whichever package ran alone.
  The finding's list of processes working in the directory is there to expose that case.
- Each binary opens two inotify instances.
  This machine allows 512 per user, and a watch that cannot open falls back to the check after the run.

## Tests

- `TestCheckNewGitEntries` covers these cases:
  - an entry seen alone fails the package;
  - an entry seen beside other binaries keeps the code and names them;
  - the processes working in the directory are named;
  - an unreadable process table and an entry no watch saw both keep the code;
  - an entry seen but already removed is still reported;
  - the existing cases for keeping a directory, a gitfile and a symlink, and for an entry that existed before the run.
- `TestRunCheckingGitEntries` creates a `.git` during a fake run and checks that the real `fswatch` watch takes a snapshot of the right directory.
- `TestSnapshotProcesses_NamesAChildWorkingInTheDirectoryAndLeavesOutThisBinary` (tier1, Linux) reads the live process table.
  It checks that a child started in the directory is named and that the test binary itself is never counted as another binary.
- `TestClassifyProcesses` covers which processes count as other test binaries, which count as descendants, and which work in the directory.
