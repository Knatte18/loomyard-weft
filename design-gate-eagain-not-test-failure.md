# lym gate test reports transient fork/thread exhaustion as test failures

`gate-eagain-not-test-failure` · Note · bug, gate

Imported from [issue #887](https://github.com/Knatte18/loomyard/issues/887).

## What happened

Two `lym gate test --wait` runs failed with `fork/exec ... resource temporarily unavailable` and `runtime: failed to create new OS thread (errno=11)`.
A `go vet` run and one PreToolUse hook failed the same way ("Cannot fork").
The failures hit packages the change did not touch.
`ulimit -u` was far above the live thread count, and a rerun a minute later was green.
The cause looks like short-lived process or thread pressure from other runs sharing the machine.

## Why it is abnormal

The gate surfaces these as ordinary build or test failures.
The cause is an environment error that a retry clears, and reporting it as a test failure hides that.

## What to do

Have `lym gate test` recognise fork/exec EAGAIN and thread-creation failures and report them as a retryable environment error, or retry them automatically.
Consider limiting how many gate runs execute concurrently on one machine.

Provenance: task `ide-seed-current-worktree`, note `Darn-Write.md`.
