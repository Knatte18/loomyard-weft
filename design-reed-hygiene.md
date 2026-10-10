# Reed hygiene: daemon log and socket path, plain strand shells, watchdog child CPU

`reed-hygiene` · Task · bug, enhancement, reed · running · Finalize

## Goal

Three reed fixes: daemon log and socket path, strand panes that read the user's tmux config, and the watchdog's idle child CPU.

## 1. Daemon Debug lines and socket path (#495)

Found while measuring the daemon's idle behaviour in task `daemon-wakeups`:

- The daemon's idle Debug spawn lines reach no sink, so spawn counts could not be read from lyx logs; the agent counted spawns through a tmux wrapper.
  Expected: those Debug lines go to the daemon log at a verbosity the operator can enable.
- A tmux socket path under a deeply nested scratch directory passed the 108-byte socket limit, and `TMUX_TMPDIR` had to be set by hand.
  Expected: lyx detects an over-long socket path and fails with a message naming `TMUX_TMPDIR`, or chooses a short default location.

## 2. Strand panes read `~/.tmux.conf` and start login shells

`test-suite-weight` (b7d4fd0ba) moved the test kit to non-login panes without the user's tmux config; reed's own servers still read `~/.tmux.conf` and start login shells for strand panes, recorded as a bound in PATTERN.
Expected: reed's servers and strand panes start without the user's tmux config and without login shells, and the PATTERN bound is lifted.

## 3. Watchdog children burn idle CPU

From `daemon-wakeups` (`docs/benchmarks/daemon-idle-cpu.md`): `lyx reed watchdog` spawns `list-sessions` far less often since that run (11.8 to 1.6 per minute), but idle CPU stayed at about 100 s per hour, about 85 s of it in child processes the benchmark shim cannot see.
Expected: find which processes the watchdog's children are (likely the idle sessions' own watchers), bring them under `PATTERN-daemon-wakeups`, and re-measure.
