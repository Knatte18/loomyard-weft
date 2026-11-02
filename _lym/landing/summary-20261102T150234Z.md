# Let tmuxkit start under /dev/shm with a ten-digit uid, then measure it

## What changes

`internal/lymtemp` now names a run directory `r` plus four random lower-case letters or digits, instead of `r` plus six hex digits.
With `TMPDIR=/dev/shm` and a ten-digit uid, `tmuxkit.Main`'s longest socket path, `/dev/shm/lym-<uid>/r????/tmux-<uid>/<61-byte key>`, was 109 bytes.
It is now 107, at the `sun_path` limit, so a package whose `TestMain` calls `tmuxkit.Main` starts there.

Every path that was valid before stays valid, since only the run directory's name gets shorter.
The sweep and `resolveRootPath` match run directories by the `r` prefix alone, so directories with the old names are still swept.
The name space drops from about 16.8 million to about 1.7 million names, which leaves the retry bound of 16 draws plenty of room.

## Why this cut

Three bytes of the socket path could move:

- The `lym-<uid>` root and the `tmux-<uid>` directory that tmux makes are shared with production users of the temp root (creel's socket, `lym`'s own run directory), and tmux fixes the second.
- The 61-byte key cap is reed's `ServerName` limit, which a fixture hub's key can reach, so shortening it would change reed.
- The run directory name belongs to `lymtemp` alone, and nothing parses it beyond its prefix.

The run directory name was the one cut that touches only `lymtemp`.

## Tests

- `TestCheckSocketPath` gains a row for a ten-digit uid under `/dev/shm`, built from the name of the run directory `lymtemp.Dir` actually made, so it fails if the name grows again.
- `TestNewRun_HoldsItsLockUntilRemoved` pins the new name shape.

## Docs

- The `lymtemp` package doc and `pattern/PATTERN-lym-temp-root.md` describe the new name.
- `docs/benchmarks/test-suite-timing.md` gains the section `tmuxkit-short-socket-path`, the paired measurement that `fabricengine-shm-bench` could not make.
  Every `/dev/shm` run passed, and the files took 35 MB.
  In the two pairs that ran concurrently, the shm run was 2% and 7% faster, so `/dev/shm` does not avoid the Cortex XDR slowdown.
  One of the three pairs did not run concurrently, because another worktree held a gate slot, and the report says so.
