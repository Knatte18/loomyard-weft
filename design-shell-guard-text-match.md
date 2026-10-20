# Shell guard refuses read-only commands that merely name a refused go-run tier

`shell-guard-text-match` · Note · bug, claude

Imported from [issue #770](https://github.com/Knatte18/loomyard/issues/770).

**What happened**
The shell guard that refuses raw `tmux`-tier `go test` runs matches the command text anywhere.
A plain `grep` for the tier's flag string in test files, and a heredoc that drafted test code containing that string, were both refused as if they were test runs.

**Why it is abnormal**
Neither command runs a test.
The guard blocks legitimate reads and file drafting, and the agent has to search with a pattern that avoids the literal.

**What to do**
Match the guard on the command's executable and arguments (a real `go test` invocation), not on text inside grep patterns, heredoc bodies or quoted arguments.

Provenance: task `gate-fewer-new-binaries`, note `Darn-Write.md`.
