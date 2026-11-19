# tmuxkit: fail a package for a stray .git only on its own process

`tmuxkit.Main` checked for a `.git` that appeared in a shared temp directory (the lym temp root or the system temp directory) and failed every package that ran at the same time.
A fixer then spent a round on a failure its task did not cause, as with `/tmp/.git` failing `shedverbs` and `shedcli` in an unrelated run.

The check now blames a package only when its own binary, or a descendant of it, had its working directory in that directory when the entry appeared.
Any other sighting is a machine finding: it keeps the package's exit code and names the other test binaries running at that moment and every process working in the directory.
Those binaries are candidates, not the writer.
The writer of the `/tmp/.git` seen on 2026-11-18 is still not identified; the finding is meant to name it the next time it appears.

## What changes

- The check moves from `tmuxkit.go` into a new `internal/testkit/tmuxkit/gitentry.go`.
  `Main` watches both shared directories through `internal/fswatch` for the creation of a `.git`.
  On Linux it reads `/proc` at that moment and classifies each process as one of this binary's own, another test binary, or one working in the directory.
- An entry that exists after the run without a sighting, because no watch could be opened, is reported as a machine finding and never fails the package.
- Every sighting of an entry is reported.
  An empty directory is still removed, and any other entry is named for removal by hand.
- `internal/fswatch/testmain_test.go` becomes an external test package (`fswatch_test`), because `tmuxkit` now imports `fswatch` and an in-package test file could not import `tmuxkit` back.
- `pattern/PATTERN-test-isolation.md` and the `tmuxkit` package doc describe the new rule and its bounds.

## Design decisions

- Blame needs evidence of writing.
  Mere presence of the entry, or a concurrent binary, is not enough to fail a package.
- The check never fails a package off Linux, when the process table does not hold the binary itself, or when no watch saw the entry appear.
  A writer that exits before the snapshot, or is no test binary (a daemon a finished test left, for example), fails no package.
- A finding quotes at most 200 characters of each process's argv, so a long command line cannot flood the output.

## To check by hand

- The writer of the stray `.git` is not found or fixed; the test that writes outside its own `t.TempDir` is still open.
  Watch the next machine finding to learn who wrote it.
- `blame-without-evidence-of-writing` (`internal/testkit/tmuxkit/gitentry.go`, `checkNewGitEntries`): a sighting with `ownInDir` blames the package from the working directory alone, which does not prove that process created the entry.
- `classify-processes-test-lacks-keep-directive` (`internal/testkit/tmuxkit/gitentry_test.go`): the test of `classifyProcesses` lacks the project's keep directive.
- `finding-text-claims-common-binary-is-writer` (`internal/testkit/tmuxkit/gitentry.go`, `describeGitEntry`): the machine-finding text says "the writer is one of them, this package or a process that is no test binary", which reads as certain although the cause is not established.
- The Linux snapshot test, `tmuxkit_snapshot_linux_test.go`, reads the live process table and was not run on other platforms; `tmuxkit_snapshot_other_test.go` covers the unattributed case there.
