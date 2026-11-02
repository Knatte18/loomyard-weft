# gitexec TestRun_RemoteBounds fails under machine load

`gitexec-remote-bounds-load-flaky` · Task · bug, git, gate · running · Record-Write

## Seen (orch, 2026-11-02, from the darn writer of `tmuxkit-short-socket-path`)

`gitexec` `TestRun_RemoteBounds/a_remote_that_never_answers...` sets a 200 ms stall bound.
At a load average of about 30, git is killed before its ext script writes its count file, and the test fails.
`heddle-render-ignores-anchor` failed the same way (gate log `20261102T135456Z-tier1-2983004`), and a rerun at `-count=1` by the writer of `tmuxkit-short-socket-path` failed the same way, although its change touches neither package.

## Idea

Make the test's bound independent of git start-up time, for example wait for the count file before the stall starts counting, rather than raising the number.
The writer reads the test and says what its 200 ms is meant to prove.
