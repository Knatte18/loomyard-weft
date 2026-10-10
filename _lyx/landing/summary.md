# Reed hygiene: Debug trace, socket check, hermetic servers, poll backoff

Four reed fixes that land together.

Closes #495

## What changes

- **Debug trace is reachable.**
  The durable logger sink admits Debug records when the process verbosity is Debug (`-vv` or `LYX_LOG_LEVEL=debug`).
  The default stays Info and above, and the sink's 8 MiB cap bounds a chatty trace.
  The CLI root hook keeps the `LYX_LOG_LEVEL` level when no `-v` is given.
- **The watchdog verb keeps its stderr half.**
  `lyx reed watchdog` no longer discards it, so a foreground `-vv` run writes Debug lines to stderr or `LYX_LOG_FILE`.
  `SpawnWatchdog` drops `LYX_LOG_FILE` from the daemon's environment and passes `LYX_LOG_LEVEL` on, so a detached daemon never writes an uncapped file.
  The verb's `Long` help names the two routes to a daemon Debug trace and says a running daemon keeps its start-up level.
- **Over-long socket paths are refused.**
  On POSIX the boot computes the socket path as tmux does, with `TMUX_TMPDIR` symlinks resolved, and refuses a path over 107 bytes on Linux or 103 elsewhere.
  The message names the path, its length, the limit and `TMUX_TMPDIR`.
  Unresolvable `shell` values are refused before any tmux round trip, naming the key.
  `Up`'s zoom bracket runs the same host pre-flight (`preflightBootHost`), since it issues a tmux call before the boot refusals.
- **Reed's servers are hermetic.**
  On POSIX the server-starting invocation and the capability probe carry `-f /dev/null`, and each session pins `default-shell` and `default-command` to the resolved `shell`.
  Strand panes therefore start non-login shells and reed's servers no longer read `~/.tmux.conf`.
  `PATTERN-test-isolation`'s bound is replaced, and the test kit's tmux config text is exported as `ConfigText`.
- **Idle watchdog children are identified and backed off.**
  The daemon's per-session engines now drive the daemon's told `--tmux` binary, so a benchmark shim counts the watchers' calls.
  A fresh boot installs the window-resized clear and signal entry, so sessions with no placed strand promote out of poll mode.
  Poll mode in the resize watch loop backs off from 2 s to a 60 s ceiling (`NextWakeCadence`), which brings it under `PATTERN-daemon-wakeups`; `tickerPeriodFor` is deleted.
  `docs/benchmarks/daemon-idle-cpu.md` gains an after-hygiene row and a section naming the children.

## Design decisions to know

- The signal entry is installed in the fresh-boot path, not inside `pinGeometryOptionsLocked`, because the attach pre-flight also runs that function and a zero-pin rebuild there would wipe the session's resize pins.
- The shell pins are per-session, never `-g`, because one server serves every worktree of the hub.
  The attach pre-flight re-resolves the shell and, when it no longer resolves, warns and skips the two pins.
- The daemon's `--tmux` is the value from the worktree whose spawn started it; other worktrees' `tmux` values are ignored and not compared.
- Windows keeps its current boot, pane start and poll-only watcher; the socket check, `-f`, shell pins and hook install are POSIX-only.
- Not done: a config-driven file log level (`logger-file-level`, #234) and the spawn audit log (`spawn-audit-log`, #231).

## Behaviour to be aware of

- An operator who tuned reed's servers through `~/.tmux.conf` (prefix, key bindings, `escape-time`, terminal overrides) loses those settings, and no `reed.yaml` key replaces them.
  A server an older lyx started keeps the operator's config until it exits.
- A resize while a backed-off poll watcher waits heals up to a minute late: always on Windows, and on POSIX when a watcher never left poll mode.
- Strand panes' shells run rc files but not the login profile.

## Check by hand

- Run `lyx -vv reed watchdog` in the foreground and confirm Debug spawn lines on stderr.
  Then run `lyx reed up` with `LYX_LOG_LEVEL=debug` while no daemon runs for the hub, and confirm Debug lines in the hub logs directory's trace file.
  The fix that makes `LYX_LOG_LEVEL` survive the root hook was added late and the end-to-end path was not run by hand.
- The benchmark's counts are consistent with the watcher-round-trips hypothesis but do not confirm it; the report is a single sample per row.
- `d6` socket-path resolution (`TMUX_TMPDIR` as a short symlink to a long directory, macOS `/tmp` resolving to `/private/tmp`): review the resolution fallback in `internal/reedengine/socketfile.go`; only Linux was run.
- Nothing was run on Windows or macOS.
