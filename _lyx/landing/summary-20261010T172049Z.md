# reed: reachable Debug trace, socket-path refusal, hermetic servers

Closes #495

Three reed fixes, plus the idle-CPU investigation of the watchdog's children.

## What changes

- **Debug trace.**
  The durable log sink in `internal/logger` admits Debug records when the process verbosity is Debug (`-vv` or `LYX_LOG_LEVEL=debug`).
  Info and above stay admitted as before, and the 8 MiB cap bounds a chatty trace.
  The CLI root hook in `cmd/lyx` no longer calls `SetVerbosity` when no `-v` is given, so a level set by `LYX_LOG_LEVEL` is no longer reset to the default.
- **Watchdog verb.**
  `lyx reed watchdog` no longer discards its stderr half, so a foreground `-vv` run shows its Debug lines.
  `SpawnWatchdog` drops `LYX_LOG_FILE` from the daemon's environment, so a detached daemon never writes an uncapped file, and it passes `LYX_LOG_LEVEL` on.
  The verb's `Long` help names the two routes to a Debug trace: a foreground `-vv` run, or `lyx reed up`, `attach` or `resume` with `LYX_LOG_LEVEL=debug` while no daemon runs for the hub.
  A running daemon keeps the level it started with.
- **Told tmux binary.**
  The daemon's per-session engines drive the `--tmux` path the daemon was told, not each worktree's `reed.yaml` value.
  One binary makes the daemon's spawn set measurable through one seam.
- **Socket path.**
  On POSIX the boot refuses a tmux socket path over the OS limit (107 bytes on Linux, 103 on other POSIX systems), computed as tmux does, with `TMUX_TMPDIR` symlinks resolved.
  The message names the path, its length, the limit and `TMUX_TMPDIR` as the way forward.
  Before, tmux failed on every call and the boot loop read each failure as a server not up yet.
  `Up`'s zoom bracket in `internal/reedengine/zoom.go` now runs the same host pre-flight, since it issues a tmux call before the boot refusals.
- **Hermetic servers.**
  On POSIX, reed's server-starting `new-session` and its capability probe carry `-f /dev/null`, so reed's servers no longer read `~/.tmux.conf`.
  Each session pins `default-shell` and `default-command` to the resolved absolute `shell`, per session and never `-g`, so strand panes start non-login shells.
  An unresolvable `shell` is refused at boot naming the key; the attach pre-flight logs a warning and skips the two pins.
  `pattern/PATTERN-test-isolation.md` replaces its bound on this, and `tmuxkit` exports `ConfigText` so a `reedengine` test can pin the shell lines the kit and reed share.
- **Poll-mode backoff.**
  The resize watch loop's poll mode waits from a re-armed timer: two seconds at base, doubling to a 60-second ceiling across ticks that applied nothing and saw no box change, back to base on an applied layout, a changed box or a deferral.
  `PATTERN-daemon-wakeups.md` names this.
  `tickerPeriodFor` is deleted.
- **Resize signal entry at boot.**
  A fresh POSIX boot with `watchdog: on` installs the window-resized hook array as a zero-pin rebuild, so sessions with fewer than two panes or no placed strand leave poll mode.
  The call sits in the boot path, not in `pinGeometryOptionsLocked`, because the attach pre-flight also runs that function and a degrading attach would wipe the session's resize pins.
- **Benchmark.**
  `docs/benchmarks/daemon-idle-cpu.md` gains an after-hygiene row: children CPU fell from 14.44 s to 0.41 s per 600 s, and all three idle sessions reached signal mode.
  The per-subcommand counts are consistent with the watcher round trips being the children, but cannot confirm it, since the earlier rows' shim could not see them.

## Design decisions

- No config key and no new log artifact is added; `logger-file-level` (#234) and the spawn audit log (#231) stay open.
- Windows keeps its current boot, pane start and poll-only watcher; psmux support for `-f`, `set-hook` and `run-shell` is unverified.
- The daemon's `--tmux` is the value of the worktree whose spawn started it.
  Nothing compares it with other worktrees' values, since a config value and a resolved path to the same binary differ as text.
- `-f /dev/null` takes effect at a hub server's next start; a server an older lyx started keeps the operator's config, and its new sessions get only the shell pins.

## To check by hand

- **Lost `~/.tmux.conf` settings.**
  An operator who tuned reed's servers through `~/.tmux.conf` (prefix, key bindings, `escape-time`, terminal overrides) loses them, and no `reed.yaml` key replaces them.
- **Late resize heal.**
  A resize while a backed-off poll watcher waits heals up to a minute late: always on Windows, and on POSIX when a watcher never left poll mode.
- **Windows.**
  No Windows host ran the changes; the poll backoff is the only change that applies there.
- **Benchmark binary.**
  The benchmark ran on the branch at `c6c54c6f3`, before the root-hook fix and later review fixes, and each row is a single sample.
  Re-run the procedure on the landed binary if the figures matter.
- **Debug trace routes.**
  Confirm `LYX_LOG_LEVEL=debug lyx reed up` against a fresh hub writes Debug spawn lines to the hub's durable log, as the help text claims.
