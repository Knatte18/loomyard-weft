# Shorten lymtemp run names so tmuxkit starts under /dev/shm

`TMPDIR=/dev/shm` made `tmuxkit.Main` refuse to start for a ten-digit uid.
`checkSocketPath` computed 109 bytes for `/dev/shm/lym-1856216086/r49ee2f` plus `tmux-<uid>` and the longest key, over the 107-byte `maxSocketPath`.
This change shortens the run directory name so the same path is 107 bytes and the package starts, then records the `internal/fabricengine` `tier1` measurement that was blocked on it.

## What changes

- `internal/lymtemp` names a run directory `r` plus four random lower-case letters or digits instead of `r` plus six hex digits.
  The doc comment in `internal/lymtemp/doc.go` and `pattern/PATTERN-lym-temp-root.md` state the new form.
- `internal/lymtemp/run_test.go` matches the new name shape.
- `TestCheckSocketPath` in `internal/testkit/tmuxkit/tmuxkit_test.go` gains a case for a ten-digit uid under `/dev/shm`, built from the name of the live run directory, so it fails if the name grows again.
  The widest-uid case uses a four-character name.
- `docs/benchmarks/test-suite-timing.md` gains the section `tmuxkit-short-socket-path`, the paired `/dev/shm` against default measurement of `internal/fabricengine` `tier1`.

## Design decisions

- **Cut chosen: the run directory name.**
  `tmux-<uid>` is made by tmux itself under `TMUX_TMPDIR`, and the key length `maxKeyBytes` is the socket name's own budget, so neither is ours to trim cheaply.
  Two of the cut sizes follow from the path arithmetic: four characters is the longest name that fits under the limit for this uid and root, and no path that was valid before becomes invalid, since the new names are shorter.
- **Collision space shrinks.**
  Four base-36 characters give about 1.7 million names against about 16.7 million before.
  `newRun` still retries a taken name up to `runNameAttempts` and then refuses naming the root; check that this bound is enough for the number of live run directories one root holds.
- **No symlink, no config change, no gate change.**
  A symlink to shorten the path fails 75 fabric tests on path identity, so the path is real.

## Measurement result

All three `/dev/shm` runs passed the same 1600 tests as the default runs, and the machine's `/dev/shm` peaked at 509 MB.
`/dev/shm` does not avoid the Cortex XDR slowdown: in the two pairs that ran side by side the shm run was 2% and 7% faster.
Pair 2 measured nothing, because a busy gate slot kept its two runs from overlapping; the report says so.
The report's note says a permanent `/dev/shm` setting is not worth a task, and names the cost of the processes the tests start as the lead to test next.

## To check by hand

- Pair 2 of the measurement is unusable; the conclusion rests on pairs 1 and 3 only.
  Rerun the pair if a third data point matters.
- The measurement ran once on one machine (OSL-1046); it was not rerun on any other.
- `lym gate test` was run for `internal/fabricengine` `tier1` only; no other package's tests were rerun under `TMPDIR=/dev/shm`.
