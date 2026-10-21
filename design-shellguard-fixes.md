# Shell guard: match test runs on the command, refuse heredoc writes to tracked files

`shellguard-fixes` · Task · bug, claude · running · Describe

## Goal

The shell guard (`internal/shellguard`) refuses what it is meant to refuse and nothing else: a refused test run is matched on the command that runs, and a scripted write to a tracked file is refused like the other edit-directive cases.

## Text match refuses commands that run no test

Seen ([issue #770](https://github.com/Knatte18/loomyard/issues/770)): the guard that refuses raw `tmux`-tier `go test` runs matches the command text anywhere.
A plain `grep` for the tier's flag string in test files, and a heredoc that drafted test code containing that string, were both refused as if they were test runs, so the agent had to search with a pattern that avoids the literal.

Expected: the guard matches on the command's executable and arguments, a real `go test` invocation, not on text inside grep patterns, heredoc bodies or quoted arguments.

## Heredoc writes slip past the edit directive

Seen ([issue #772](https://github.com/Knatte18/loomyard/issues/772)): an agent appended a section of a markdown doc through a shell heredoc instead of Edit or Write, against the prompt's edit directive.
The guard did not refuse it, while it refused other commands in the same session, so enforcement is uneven.

Expected: the guard refuses heredoc and redirect writes (`cat >> file <<EOF`, `> file`) to tracked files under the edit directive.
Writes to untracked scratch paths, such as `.scratch/`, stay allowed, since the directive governs project files.

Provenance of both: task `gate-fewer-new-binaries`, note `Darn-Write.md`.
