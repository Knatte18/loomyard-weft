# Let tmuxkit start under /dev/shm with a ten-digit uid, then measure fabricengine tier1 there

`tmuxkit-short-socket-path` · Task · enhancement, testkit, cwd, gate, fabric

## Seen (orch, from the `fabricengine-shm-bench` report, 2026-11-02)

On OSL-1046 the uid is ten digits (a domain account), and `TMPDIR=/dev/shm` makes `tmuxkit.Main` refuse to start any package that has a `TestMain` calling it, `internal/fabricengine` among them.
`checkSocketPath` in `internal/testkit/tmuxkit/tmuxkit.go` computes the longest socket path as the run directory, `tmux-<uid>` and the longest key, and the result for `/dev/shm/lym-1856216086/r49ee2f` is 109 bytes, over the 107-byte limit (`maxSocketPath`).
`/dev/shm` is the shortest in-memory path there is, so no `TMPDIR` on this machine puts the tests in memory.
The report is the section `fabricengine-shm-bench` in `docs/benchmarks/test-suite-timing.md`; it measured the default run only (351.85 s, passed) and one failed run through a symlink, so the question below is still open.

## Why it matters

`internal/fabricengine` takes 348.72 s of the `tier1` wall on this machine against 45.7 s on a machine without Cortex XDR, and the working guess is that Cortex scans every file the tests create on disk.
The guess cannot be tried until the tests can run from `/dev/shm`.
A symlink that shortens the path was tried and fails 75 fabric tests on path identity (ownership checks compare the path a test was handed with the path git reports), so the fix has to give a real short path, not a link.

## Part 1: the fix

Make a package using `tmuxkit.Main` start with `TMPDIR=/dev/shm` for a ten-digit uid, so the longest socket path is within the limit.
The writer reads where the bytes go before choosing: the run directory `lym-<uid>/r<hex>` comes from `internal/lymtemp`, `tmux-<uid>` is the directory tmux itself makes under `TMUX_TMPDIR`, and the key length is `maxKeyBytes`.
It picks the cut that costs least and keeps the other temp-root users working, for example a shorter run directory name, a socket directory outside the uid-long root, or a shorter key, and says why it chose it.
A path that was valid before must stay valid.
Add a test for a ten-digit uid under a `/dev/shm`-length root.

## Part 2: the measurement the earlier task could not make

When the fix is in, run the measurement `fabricengine-shm-bench` set out, on the same machine.
The question is two things and only these: can `internal/fabricengine` `tier1` run with its temp root on `/dev/shm` (do the tests pass, does memory hold), and does that avoid Cortex's scan, shown by the package's wall under `/dev/shm` against the default under the same machine load.
How near it comes to the Ryzen machine's absolute time is no part of the question.

- Run only `internal/fabricengine` in `tier1`, with the test binary cache bypassed (`-count=1`), through `lym gate test`: `lym gate test --wait --tier 1 ./internal/fabricengine -- -count=1`.
  The setting is `TMPDIR=/dev/shm` for the run only, no symlink and no config change; check that files land in `/dev/shm` by sampling `find /dev/shm/lym-<uid> -type f` and `du` during the run, as the earlier report did.
- Three pairs, default temp root against `/dev/shm`, the two runs of a pair started together so both meet the same load.
  Each pair needs two gate slots at once; the two runs must be started in one command that backgrounds both, since the earlier attempt ran them one after the other and its pair ratio measured nothing.
  Alternate which setting starts first in each pair.
  Only the ratio of a pair counts, since two runs slow each other.
- Read each run's package wall from the test output and `/dev/shm`'s use before and after each shm run.
  When `/dev/shm`'s use passes 6 GB, stop the pair's runs, report the peak and go on to the report.
- Record each run's load average at start and end, the revision and the Go version.
- At most six runs: three default and three `/dev/shm`, no cold run outside them.
  If the shm runs fail for a reason other than the socket path, stop, report what failed and run no more: fixing a second cause is a task of its own, filed as a note.

The report is a new section in `docs/benchmarks/test-suite-timing.md` after `fabricengine-shm-bench`, with the six walls by pair, the ratio of each pair, the memory use, the load averages, whether every run passed, and a plain statement of whether `/dev/shm` can be used and whether it avoids the Cortex slowdown.
It lands in the same commit as the fix.
The task makes no config change and no gate change; whether a permanent `/dev/shm` setting is worth a task goes in the report as a note.
