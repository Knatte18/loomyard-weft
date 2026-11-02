# Let tmuxkit start under a short TMPDIR with a ten-digit uid

`tmuxkit-short-socket-path` · Task · enhancement, testkit, cwd, gate · draft

## Seen (orch, from the `fabricengine-shm-bench` report, 2026-11-02)

On OSL-1046 the uid is ten digits (a domain account), and `TMPDIR=/dev/shm` makes `tmuxkit.Main` refuse to start any package that has a `TestMain` calling it, `internal/fabricengine` among them.
`checkSocketPath` in `internal/testkit/tmuxkit/tmuxkit.go` computes the longest socket path as the run directory, `tmux-<uid>` and the longest key, and the result for `/dev/shm/lym-1856216086/r49ee2f` is 109 bytes, over the 107-byte limit (`maxSocketPath`).
`/dev/shm` is the shortest in-memory path there is, so no `TMPDIR` on this machine puts the tests in memory.
The report is the section `fabricengine-shm-bench` in `docs/benchmarks/test-suite-timing.md`.

## Why it matters

`internal/fabricengine` takes 348.72 s of the `tier1` wall on this machine against 45.7 s on a machine without Cortex XDR, and the working guess is that Cortex scans every file the tests create on disk.
The guess cannot be tried until the tests can run from `/dev/shm`.
A symlink that shortens the path was tried and fails 75 fabric tests on path identity (ownership checks compare the path a test was handed with the path git reports), so the fix has to give a real short path, not a link.

## Task

Make a package using `tmuxkit.Main` start with `TMPDIR=/dev/shm` for a ten-digit uid, so the longest socket path is within the limit.
The writer reads where the bytes go before choosing: the run directory `lym-<uid>/r<hex>` comes from `internal/lymtemp`, `tmux-<uid>` is the directory tmux itself makes under `TMUX_TMPDIR`, and the key length is `maxKeyBytes`.
It picks the cut that costs least and keeps the other temp-root users working, for example a shorter run directory name, a socket directory outside the uid-long root, or a shorter key, and says why it chose it.
A path that was valid before must stay valid.
Add a test for a ten-digit uid under a `/dev/shm`-length root.

## After it lands

The operator reruns the `fabricengine` `tier1` measurement with `TMPDIR=/dev/shm`, paired against the default and started together, as the report's note says; that rerun is a bench of its own and no part of this task.
The task makes no config change and no gate change.
