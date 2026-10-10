# reed: Debug trace, socket-path refusal, hermetic servers, poll backoff

## What changes

- **Debug trace.**
  The durable log sink admits Debug records when the process verbosity is Debug (`-vv` or `LYX_LOG_LEVEL=debug`); the default stays Info and above, and the 8 MiB cap still bounds the file.
  The root command's pre-run hook only calls `SetVerbosity` when `-v` was given, so `LYX_LOG_LEVEL` is no longer reset to Warn before a verb runs.
- **Watchdog logging.**
  `lyx reed watchdog` no longer discards its stderr half, so a foreground `-vv` run shows its Debug lines.
  `SpawnWatchdog` drops `LYX_LOG_FILE` from the daemon's environment and keeps `LYX_LOG_LEVEL`, so a detached daemon never writes an uncapped file.
  The verb's help names the two routes to a daemon trace; a daemon already running keeps its start level.
- **Told tmux binary.**
  The daemon's per-session engines use the daemon's `--tmux` path, not each worktree's `reed.yaml` value.
  A different `tmux` value in another worktree is ignored silently, by design.
- **Socket path.**
  On POSIX, boot computes the socket path the way tmux does (resolved `TMUX_TMPDIR` or `/tmp`, `tmux-<uid>`, the `-L` key) and refuses one over 107 bytes on Linux or 103 elsewhere.
  The message names the path, its length, the limit and `TMUX_TMPDIR`.
  Windows skips the check.
  `Up`'s zoom bracket runs the same host pre-flight (`preflightBootHost`), since it issues a tmux call before the boot refusals.
- **Hermetic servers.**
  On POSIX, the server-starting invocation and the capability probe carry `-f /dev/null`, so reed's hub servers read no `~/.tmux.conf`.
  Each session pins `default-shell` and `default-command` to the resolved `shell`, per session and never with `-g`, so strand panes start non-login shells.
  An unresolvable `shell` is refused at boot naming the key; the attach pre-flight warns and skips those two pins.
  `PATTERN-test-isolation` replaces its old bound, and the test kit exports its config text (`ConfigText`) so a `reedengine` test pins the shared shell lines.
- **Resize watchers.**
  A fresh boot on POSIX with `watchdog: on` installs the window-resized clear and signal entry, so sessions with fewer than two panes promote out of poll mode.
  The install is in the boot path, not in `pinGeometryOptionsLocked`, because a degrading attach would otherwise wipe the session's resize pins.
  Poll mode now backs off from 2 s to a 60 s ceiling through `NextWakeCadence`, as `PATTERN-daemon-wakeups` requires; `tickerPeriodFor` is gone.
- **Benchmark.**
  `docs/benchmarks/daemon-idle-cpu.md` gains an after-hygiene row and a section on the children.
  Its counts are consistent with the hypothesis that the children are mostly poll-mode round trips, but do not confirm it.

Closes #495

## Design decisions to know

- A config-driven file log level (`logger-file-level`) and a spawn audit log are out of scope; no config key is added.
- Windows boot, pane start and poll-only watching are unchanged.
- `-f /dev/null` is a behaviour change for operators who tuned reed's servers through `~/.tmux.conf`: prefix, key bindings and `escape-time` are lost, and no `reed.yaml` key replaces them.
  A server started by an older lyx keeps that config until it exits.
- Strand-pane shells run their rc files but no longer the login profile.
- A resize while a backed-off poll watcher waits heals up to a minute late: always on Windows, and on POSIX when a watcher never left poll mode.
- The `cmd/lyx` and other test-budget rows are raised in `cmd/lyx/testdata/test-budget.yaml`.

## To check by hand

- Run `lyx -vv reed watchdog` in the foreground and confirm Debug lines on stderr, and `LYX_LOG_LEVEL=debug lyx reed up` with no daemon running and confirm Debug lines in the daemon's durable trace; the help text and docs claim both.
- Set a long `TMUX_TMPDIR` (including a short symlink to a long directory) and confirm the boot refusal message.
- Boot a hub with a marker in `~/.tmux.conf` and confirm the marker is unset on reed's server and a strand pane's shell is not a login shell.
- The benchmark rows are single samples taken in a scratch hub; the shell-pin and `-f` behaviour on macOS and Windows was not run.
