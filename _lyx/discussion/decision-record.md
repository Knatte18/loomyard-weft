# Decision record — reed-hygiene

## Goal

Three reed fixes, landing as one task and closing #495:
the watchdog daemon's Debug lines reach a log the operator can switch on, an over-long tmux socket path is refused with `TMUX_TMPDIR` named as the way forward,
reed's own tmux servers stop reading `~/.tmux.conf` and strand panes stop starting login shells,
and the idle CPU the watchdog's children burn is traced to the per-session resize watchers, brought under `PATTERN-daemon-wakeups` and re-measured.

## Scope

In:

- `internal/logger`: the durable trace sink admits Debug records when the process verbosity is Debug.
- `internal/reedcli`: `lyx reed watchdog` stops discarding its stderr half; the daemon's per-session engines drive the tmux binary the daemon was told.
- `internal/reedengine`: the socket-path check at boot; `-f /dev/null` on the server-starting invocation plus per-session `default-shell` and `default-command` pins on POSIX; the resize signal entry installed at session boot; poll-mode backoff in the resize watch loop.
- `pattern/PATTERN-test-isolation.md`: the login-shell and `~/.tmux.conf` bound is replaced; `pattern/PATTERN-daemon-wakeups.md` names the poll mode's backoff.
- `docs/benchmarks/daemon-idle-cpu.md`: a re-measurement after this task by the same procedure, with the children identified.
- Package docs in the touched packages' `doc.go` and package comments; the watchdog verb's `Long` help names how to get a Debug trace.

Out:

- A config-driven file log level in `logger.yaml` (board note `logger-file-level`, #234) and the spawn audit log (`spawn-audit-log`, #231): this task adds no config key and no new log artifact.
- Any change to Windows boot behaviour: Windows keeps its current server start, pane start and poll-only watcher, with the bounds recorded.
- Folding the `loomcli` and `orchcli` probe copies (`tmuxkit-probe-copies`, #277) and the status strand on llm-driven runs (`retire-loom-status-strand`, #301).
- A short default socket directory of reed's own: reed keeps resolving the socket as tmux does.

## Decisions

### D1. The durable sink admits Debug when the process verbosity is Debug

Decision: `durableHandler.Enabled` admits a Debug record when the process threshold is Debug, set by `-vv` or `LYX_LOG_LEVEL=debug`; Info and above stay admitted unconditionally, and the default stays Info and above.
The inner text handler follows the same rule.
Bound: the sink's existing 8 MiB cap with its single truncation marker bounds a chatty Debug trace; nothing else widens.

Rationale: the daemon's Debug spawn lines already exist (`PATTERN-spawn-observability` puts a polling probe's spawns at Debug), and the only reason they reach nothing is that the durable sink refuses Debug whatever the operator asked for.
Enabling them through the verbosity switch every lyx process already has costs no new config surface, and leaves `logger-file-level`'s config floor as that note's own work.

### D2. The watchdog verb keeps its stderr half

Decision: `lyx reed watchdog` no longer calls `logger.SetOutput(io.Discard)`.
It still points the durable sink at the hub logs directory first.
A detached daemon's stderr is already the null device, since `SpawnWatchdog` leaves the child's stdio nil, so nothing changes for a spawned daemon; a foreground run honours `-v`, `-vv`, `LYX_LOG_LEVEL` and `LYX_LOG_FILE` like every other verb.
The verb's `Long` help states the operator's two routes to a Debug trace of the daemon: run it in the foreground with `-vv`, or start it detached from a shell with `LYX_LOG_LEVEL=debug` set, which `SpawnWatchdog` passes on by inheriting the environment.

Rationale: the discard is what made the benchmark's `LYX_LOG_LEVEL=debug` foreground run write five warning lines and nothing else; the detached case never needed it.

### D3. The daemon's per-session engines drive the told tmux binary

Decision: `enterSession` builds each watched session's engine from the worktree's `reed.yaml` as today, with the config's `tmux` value replaced by the daemon's told `--tmux` path.
Bound: a worktree whose `reed.yaml` names a different tmux binary than the daemon was told is driven with the daemon's; since every worktree of a hub shares one server on one socket key, the binaries must agree already.

Rationale: the daemon is told its tmux and must derive nothing, yet its per-session watchers spawned the worktree config's binary, which is why the benchmark's `--tmux` shim counted the discovery loop's eight `list-sessions` and none of the children that cost 85 s per hour.
One binary makes the daemon's spawn set measurable through one seam.

### D4. The resize signal entry is installed at session boot

Decision: on POSIX with `watchdog: on`, the fresh-boot pins (`pinGeometryOptionsLocked`) install the window-resized hook array as a zero-pin rebuild, that is the clear and the signal entry alone, by calling the existing install function.
A later layout apply or attach pre-flight rebuilds the array with its pins as today.
Windows and `watchdog: off` keep their current unset path.

Rationale: today the signal entry reaches only a session that a layout apply reaches, so a session with fewer than two panes or no placed strand never promotes its watcher out of poll mode.
The benchmark's three strand-less sessions, and every session whose strands have all been removed, are that shape, and their watchers polled tmux every two seconds for the whole run.

### D5. Poll mode backs off

Decision: the resize watch loop's poll mode wakes from a re-armed timer, not a fixed ticker: the wait starts at the two-second base, doubles up to a 60-second ceiling across ticks whose re-apply issued nothing and observed no change of the live box, and returns to the base on a tick that applied a layout or saw the box change.
Dormant mode keeps its fixed minute.
Signal mode keeps no timer.
The cadence rule is the shared `NextWakeCadence`.
Bound: on Windows, which is poll-only, a resize while the watcher is backed off is healed at the next tick, up to a minute later; on POSIX poll mode is transient after D4.

Rationale: `PATTERN-daemon-wakeups` requires a wake that spawns a process to back off while nothing changes, and the poll mode was the one reed loop without it.
The per-tick re-apply is four or five tmux round trips, which at two seconds per session is the children CPU the benchmark could not see.

### D6. An over-long socket path is refused at boot

Decision: on POSIX, the pre-tmux block of the boot path computes the socket path as tmux does, `$TMUX_TMPDIR` else `/tmp`, then `tmux-<uid>`, then the `-L` key, through the existing `socketDirFromEnv`, and refuses when the path exceeds the 107 usable bytes of a Unix socket path.
The message names the path, its byte length, the limit and the way forward: set `TMUX_TMPDIR` to a shorter directory.
Windows skips the check, since psmux keeps no socket file.
Bound: the check runs on the boot path only; a verb run later under a longer `TMUX_TMPDIR` than the server booted with still gets tmux's own error.

Rationale: tmux prints "File name too long" and exits 0 on every invocation past the limit, which no probe reed has can tell from a slow boot, so the boot loop burned its whole budget before.
Refusing is the smaller change: a socket directory of reed's own would need `-S` on every client, in the attach chain and in the key bindings, and would hide the server from the operator's own `tmux -L` habit.

### D7. Reed's own servers read no `~/.tmux.conf`, and strand panes start non-login shells

Decision: on POSIX, the server-starting `new-session` argv carries `-f /dev/null` ahead of `-L`, on every spawn, since a running server ignores it; and the session pins, at a fresh boot and in the attach pre-flight, set the session's `default-shell` and `default-command` to the absolute path of the configured `shell`, resolved with `exec.LookPath` in the pre-tmux block, where an unresolvable shell is refused naming the `shell` key.
Strand panes keep their split with no trailing command; the session's non-empty `default-command` is what makes tmux start the shell without the login flag.
The first pane and Selvage keep their explicit trailing shell command.
Windows keeps its current boot, recorded as a bound.
`PATTERN-test-isolation`'s bound that reed's own servers read `~/.tmux.conf` and start login-shell strand panes is replaced by this rule; the kit's pre-started server keeps its own config, with its two extra lines, `exit-empty off` and the kit marker, and a `reedengine` test pins the shell lines the two share, since the kit cannot import `reedengine`.

Rationale: the test kit already runs every tmux-tier test this way, so the shape is proven for everything lyx types into a pane; the operator's `~/.tmux.conf` was the one input that made two reed servers differ, and reed pins every option it depends on itself.
Per-session pins rather than a reed-written config file keep each worktree's own `shell` value and need no file to manage; `-f /dev/null` is all the server start needs.

### D8. The children are identified, and the benchmark is re-run

Decision: the children the benchmark could not see are the per-session resize watchers' poll-mode round trips, issued through the worktree config's tmux binary, which the shim did not wrap, at two seconds per session; the name-repair pass, already backed off to a minute, is the rest.
The plan's last card re-runs `docs/benchmarks/daemon-idle-cpu.md`'s procedure on this task's binary, with the shim now counting every subcommand because of D3, and appends the result as a third row and a short section naming the children, keeping the document a measurement report.

Rationale: the task asks for the identification and a re-measurement; the report format already exists and a third row makes the before, after and after-hygiene figures one table.

## Constraints

- `PATTERN-daemon-wakeups`: the poll backoff is the rule applied; the loop's files are already in the guard's list, and the base stays at the one-second floor or above.
- `PATTERN-spawn-observability`: the overlay's Debug line per tmux spawn is the record D1 makes reachable; no spawn site changes level.
- `PATTERN-told-geometry` and `PATTERN-shell-mechanics-seam`: the daemon derives nothing (D3); the shell path resolved in D7 is a tmux option value, not a pane command string, so no shell token is composed.
- `PATTERN-pane-binary-resolution`: the split and the launch script are untouched; only the session option changes how the pane's shell starts.
- `PATTERN-test-isolation`: the bound is replaced in the same commit as D7; the kit's hermetic server keeps the marker option reed's own boots never set.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: fake-tmux tests stay untagged; a test that boots reed under a marker `~/.tmux.conf` is `tmux`-tier and registers its key after reed's own first boot.
- `PATTERN-refusal-way-forward` binds webster, shed and loom, not reed; the new refusals (D6, D7) still name their way forward and are reached by a test.
- Test budget: `cmd/lyx/testdata/test-budget.yaml` rows for `internal/reedengine`, `internal/reedcli`, `internal/logger` and `internal/testkit/tmuxkit` are raised in the card that needs them.
- Docs land in the same commit: `internal/reedengine/doc.go`, `internal/reedcli` package comment, the `internal/logger` package doc's level policy, the two PATTERN background files, and the benchmark report.
- The commit message carries `Closes #495`.

## Auto-mode assumptions

- No operator answered; every pick below is the producer's.
- A1: the enabling switch for Debug in the durable sink is the process verbosity, not a config key, so `logger-file-level` stays an open note.
- A2: Windows is left as it is in D4 through D7, since psmux's support for `-f`, `set-hook` and `run-shell` is unverified and no Windows host runs here.
- A3: the poll-mode ceiling is 60 seconds, the ceiling the discovery and name-repair loops already use.
- A4: the shell options are per-session pins, not a reed-written config file.
- A5: the re-measurement keeps the earlier runs' shape, three idle strand-less sessions, so the rows compare.

## Open risks

- An operator who tuned reed's servers through `~/.tmux.conf` loses those settings after D7; `reed.yaml` is the configuration surface.
- A Windows resize while a backed-off poll watcher waits heals up to a minute late (D5).
- D1 raises the volume of a `-vv` process's trace file to the cap; the truncation marker tells the reader so.
- The re-measurement is a single sample each, as the earlier rows were.

## Acceptance criteria

- A Debug record lands in the durable trace file when the process verbosity is Debug, and not at the default; Info and Warn land as before.
- `lyx -vv reed watchdog` run in the foreground writes its Debug lines to stderr or `LYX_LOG_FILE`; its durable sink still goes to the hub logs directory.
- The daemon's per-session engines report the told tmux path, whatever the worktree's `reed.yaml` says.
- A fresh boot with `watchdog: on` on POSIX issues the window-resized clear and the signal entry; `watchdog: off` issues the unset; Windows issues neither.
- The watch loop in poll mode, driven by injected timing, waits the base after an applied or changed tick and doubles to the ceiling across unchanged ticks; dormant and signal modes are unchanged.
- The boot refuses a socket path over 107 bytes with a message naming `TMUX_TMPDIR`, admits one at the limit, and skips the check on Windows; the check is a pure function of directory and key.
- The server-starting argv carries `-f /dev/null` on POSIX and not on Windows; the session pins carry `default-shell` and `default-command` with the resolved absolute shell at boot and at the attach pre-flight.
- A `tmux`-tier boot of reed under a `HOME` whose `.tmux.conf` sets a marker option leaves the marker unset on reed's server, and a strand pane's start command is the resolved shell, not a login shell.
- An unresolvable `shell` value is refused before any tmux round trip, naming the key.
- A reedengine test pins the kit's config shell lines against reed's own pins.
- `lyx loom lint-comments` passes, and the PATTERN background files reflect D5 and D7.
- The benchmark report carries a third row from a run by the recorded procedure, with the shim's per-subcommand counts.

## Notes for the plan writer

- The signal entry's one install function already documents that a zero-pin rebuild with the watchdog on installs the entry; D4 adds a call site at the fresh-boot pins, not a second writer.
- The poll timer can reuse the loop's `oneShot` and `NextWakeCadence`; the box comparison is against `lastApplied`, which the loop already tracks.
- `socketDirFromEnv` in `socketfile.go` is the one place reed spells tmux's socket directory rule; the kit holds its own 107-byte constant.
- The daemon's engine override belongs in `reedcli`, which already loads the worktree config in `enterSession`.
- The logger's package doc states "Debug never reaches the durable sink" in several places; each must follow D1.
- The measurement card is manual work in a scratch hub, as the earlier report records, never a test.
