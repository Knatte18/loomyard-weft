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
- `internal/reedengine`: `SpawnWatchdog` drops `LYX_LOG_FILE` from the daemon's environment; the socket-path check at boot; `-f /dev/null` on the server-starting invocation plus per-session `default-shell` and `default-command` pins on POSIX; the resize signal entry installed at session boot; poll-mode backoff in the resize watch loop.
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
A foreground run honours `-v`, `-vv`, `LYX_LOG_LEVEL` and `LYX_LOG_FILE` like every other verb.
`SpawnWatchdog` passes the daemon its own environment minus `LYX_LOG_FILE`, so a detached daemon's stderr half stays the null device its nil stdio gives it, whatever file the spawning shell named.
`LYX_LOG_LEVEL` is still passed on.
The verb's `Long` help states the operator's two routes to a Debug trace of the daemon: run it in the foreground with `-vv`, or, while no daemon runs for the hub, run `lyx reed up`, `attach` or `resume` with `LYX_LOG_LEVEL=debug` set, so the detached daemon that verb spawns starts at Debug.
The help also states that a daemon already running keeps the level it started with, since a new spawn finds the hub lock held and exits.
Bound: the discard's removal lets a foreground run's stderr half through, to the terminal or the `LYX_LOG_FILE` the operator set for that run; a detached daemon's stderr half reaches nothing, and its Debug trace reaches only the durable sink, under that sink's 8 MiB cap.

Rationale: the discard is what made the benchmark's `LYX_LOG_LEVEL=debug` foreground run write five warning lines and nothing else.
Without the discard, an inherited `LYX_LOG_FILE` would reroute a detached daemon's stderr half into an uncapped, unswept file for the daemon's whole life, so the spawn drops that one variable instead.

### D3. The daemon's per-session engines drive the told tmux binary

Decision: `enterSession` builds each watched session's engine from the worktree's `reed.yaml` as today, with the config's `tmux` value replaced by the daemon's told `--tmux` path.
Bound: the daemon's `--tmux` is the `tmux` value of the worktree whose spawn started it, and its watchers ignore every other worktree's `tmux` value, including a debugging wrapper one worktree names.
Nothing checks that the values agree, and the daemon accepts a difference silently: a config value and a resolved path to the same binary differ as text, so a comparison would warn on every healthy hub.

Rationale: the daemon is told its tmux and must derive nothing, yet its per-session watchers spawned the worktree config's binary, which is why the benchmark's `--tmux` shim counted the discovery loop's eight `list-sessions` and none of the children that cost 85 s per hour.
One binary makes the daemon's spawn set measurable through one seam.

### D4. The resize signal entry is installed at session boot

Decision: on POSIX with `watchdog: on`, the fresh-boot path, right after its `pinGeometryOptionsLocked` call, installs the window-resized hook array as a zero-pin rebuild, that is the clear and the signal entry alone, by calling the existing install function.
The call goes in the boot path, never inside `pinGeometryOptionsLocked`, which the attach pre-flight also runs: there a degrading attach returns before its install, and a zero-pin rebuild would wipe the session's resize pins.
The attach pre-flight keeps its current order, and a later layout apply or successful attach rebuilds the array with its pins as today.
POSIX with `watchdog: off` keeps its unset at boot; Windows issues no hook call, as today.

Rationale: today the signal entry reaches only a session that a layout apply reaches, so a session with fewer than two panes or no placed strand never promotes its watcher out of poll mode.
The benchmark's three strand-less sessions, and every session whose strands have all been removed, are that shape, and their watchers polled tmux every two seconds for the whole run.

### D5. Poll mode backs off

Decision: the resize watch loop's poll mode wakes from a re-armed timer, not a fixed ticker: the wait starts at the two-second base, doubles up to a 60-second ceiling across ticks whose re-apply issued nothing and observed no change of the live box, and returns to the base on a tick that applied a layout or saw the box change.
The other outcomes move it as follows: a deferral under a held op lock returns it to the base, since another op is changing the session; a failed re-apply doubles it, so a broken tmux costs at most one round trip a minute; a degraded box query, which is no observation, doubles it unless the tick applied a layout.
Dormant mode keeps its fixed minute.
Signal mode keeps no timer.
The cadence rule is the shared `NextWakeCadence`.
Bound: the 60-second ceiling bounds a poll-mode watcher's cost on both platforms, and a resize while it is backed off heals at the next tick, up to a minute late.
Windows is poll-only; on POSIX a watcher stays in poll mode for its lifetime when its file watch fails at promotion, and, since poll mode keeps probing for the hook, until a later attach or apply installs the hook when its hook install fails or an older lyx booted the session.
With `watchdog: off` the loop parks and never polls.

Rationale: `PATTERN-daemon-wakeups` requires a wake that spawns a process to back off while nothing changes, and the poll mode was the one reed loop without it.
The per-tick re-apply is four or five tmux round trips, which at two seconds per session is the children CPU the benchmark could not see.

### D6. An over-long socket path is refused at boot

Decision: on POSIX, the pre-tmux block of the boot path computes the socket path as tmux does: `$TMUX_TMPDIR` else `/tmp`, with its symlinks resolved and the unresolved path kept when resolution fails, then `tmux-<uid>`, then the `-L` key.
tmux measures the resolved path, so a symlinked `TMUX_TMPDIR`, or macOS's `/tmp`, which resolves to `/private/tmp`, is checked at the length tmux sees.
The directory rule stays the one in `socketDirFromEnv`; the resolution is the boot check's own, and the check refuses when the path exceeds the usable bytes of a Unix socket path: 107 on Linux, whose `sun_path` is 108 bytes, and 103 on every other POSIX system, macOS and the BSDs among them, whose `sun_path` is 104.
The message names the path, its byte length, the limit and the way forward: set `TMUX_TMPDIR` to a shorter directory.
Windows skips the check, since psmux keeps no socket file.
Bound: the check runs on the boot path only; a verb run later under a longer `TMUX_TMPDIR` than the server booted with still gets tmux's own error.

Rationale: past the limit tmux prints "File name too long" on every invocation, and the boot loop starts the server without waiting on it and reads each `has-session` connect failure as a server not up yet, so it burned its whole budget before.

### D7. Reed's own servers read no `~/.tmux.conf`, and strand panes start non-login shells

Decision: on POSIX, the server-starting `new-session` argv carries `-f /dev/null` ahead of `-L`, on every spawn, since a running server ignores it; and the session pins, at a fresh boot and in the attach pre-flight, set the session's `default-shell` and `default-command` to the absolute path of the configured `shell`.
The two shell options are set per session, never with `-g`, because one server serves every worktree of the hub and a global value would bind every sibling's strand panes to the first booter's shell.
The boot path resolves the shell with `exec.LookPath` in its pre-tmux block, where an unresolvable shell is refused naming the `shell` key.
The attach pre-flight resolves it again the same way; there an unresolvable shell logs a warning and skips the two shell pins, leaving the session's earlier values, as the pre-flight's other pins degrade.
Strand panes keep their split with no trailing command; the session's non-empty `default-command` is what makes tmux start the shell without the login flag.
A strand pane's shell therefore runs its rc files and no longer runs the login profile.
The first pane and Selvage keep their explicit trailing shell command.
Windows keeps its current boot, recorded as a bound.
Bound: `-f /dev/null` skips the operator's whole `~/.tmux.conf` on reed's per-hub servers only, every option, key binding and hook it sets, and leaves the operator's own tmux servers as they are.
The skip takes effect at a hub server's next start: a server an older lyx started keeps the operator's config until it exits, and its new sessions get only the shell pins.
Reed sets every option it depends on itself, so nothing reed needs is lost.
No `reed.yaml` key carries an operator tmux setting, and this task adds none; an operator who wants a setting on a reed server sources it by hand into the running server, where it lasts until that server exits.
`PATTERN-test-isolation`'s bound that reed's own servers read `~/.tmux.conf` and start login-shell strand panes is replaced by this rule; the kit's pre-started server keeps its own config, with its two extra lines, `exit-empty off` and the kit marker, and a `reedengine` test pins the shell lines the two share, since the kit cannot import `reedengine`.

Rationale: the test kit already runs every tmux-tier test this way, so the shape is proven for everything lyx types into a pane; the operator's `~/.tmux.conf` was the one input that made two reed servers differ, and reed pins every option it depends on itself.

### D8. The children are identified, and the benchmark is re-run

Decision: the working hypothesis, from reading the code and not yet measured, is that the children the benchmark could not see are mostly the per-session resize watchers' poll-mode round trips, issued through the worktree config's tmux binary, which the shim did not wrap, at two seconds per session, with the name-repair pass, already backed off to a minute, the remainder.
The plan's last card re-runs `docs/benchmarks/daemon-idle-cpu.md`'s procedure on this task's binary, with the shim now counting every subcommand because of D3, and appends the result as a third row and a short section naming the children, keeping the document a measurement report.
The re-run's per-subcommand counts and children CPU are what confirm or refute the hypothesis.
If they refute it, the section records the children the counts show and states that the hypothesis did not hold; the card changes no code to chase the figure.

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

## Auto-mode assumptions

- No operator answered; every pick below is the producer's.
- A1: the enabling switch for Debug in the durable sink is the process verbosity, not a config key, so `logger-file-level` stays an open note.
- A2: Windows is left as it is in D4 through D7, since psmux's support for `-f`, `set-hook` and `run-shell` is unverified and no Windows host runs here.
- A3: the poll-mode ceiling is 60 seconds, the ceiling the discovery and name-repair loops already use.
- A4: the shell options are per-session pins, not a reed-written config file.
- A5: the re-measurement keeps the earlier runs' shape, three idle strand-less sessions, so the rows compare.

## Open risks

- An operator who tuned reed's servers through `~/.tmux.conf` loses those settings after D7, such as a prefix, key bindings, `escape-time` or terminal overrides; no `reed.yaml` key replaces them, and a hand-sourced file lasts only until the server exits.
- A resize while a backed-off poll watcher waits heals up to a minute late (D5), on Windows always and on POSIX when a watcher never left poll mode.
- D1 raises the volume of a `-vv` process's trace file to the cap; the truncation marker tells the reader so.
- The re-measurement is a single sample each, as the earlier rows were.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d6-socket-path-not-realpath (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A Debug record lands in the durable trace file when the process verbosity is Debug, and not at the default; Info and Warn land as before.
- `lyx -vv reed watchdog` run in the foreground writes its Debug lines to stderr or `LYX_LOG_FILE`; its durable sink still goes to the hub logs directory.
- `SpawnWatchdog`'s child environment carries no `LYX_LOG_FILE` when the spawner's does, and carries the spawner's `LYX_LOG_LEVEL`.
- The daemon's per-session engines report the told tmux path, whatever the worktree's `reed.yaml` says.
- A fresh boot with `watchdog: on` on POSIX issues the window-resized clear and the signal entry; `watchdog: off` issues the unset; Windows issues neither.
- With `watchdog: on`, an attach pre-flight that degrades before its install leaves the session's window-resized array, pins included, untouched.
- The watch loop in poll mode, driven by injected timing, waits the base after an applied, changed or deferred tick and doubles to the ceiling across unchanged, failed and degraded-box ticks; dormant and signal modes are unchanged.
- The boot refuses a socket path over the OS's limit, 107 bytes on Linux and 103 elsewhere on POSIX, with a message naming `TMUX_TMPDIR`, admits one at the limit, and skips the check on Windows; the check is a pure function of the resolved directory, key and limit, and a `TMUX_TMPDIR` that is a short symlink to a long directory is refused at the resolved length.
- The server-starting argv carries `-f /dev/null` on POSIX and not on Windows; the session pins carry `default-shell` and `default-command`, session-scoped and never `-g`, with the resolved absolute shell at boot and at the attach pre-flight.
- An attach pre-flight whose shell no longer resolves issues no `default-shell` or `default-command` pin and still builds its argv.
- A `tmux`-tier boot of reed under a `HOME` whose `.tmux.conf` sets a marker option leaves the marker unset on reed's server, and a strand pane's start command is the resolved shell, not a login shell.
- On POSIX, a boot with an unresolvable `shell` value is refused before any tmux round trip, naming the key.
- A reedengine test pins the kit's config shell lines against reed's own pins.
- The PATTERN background files reflect D5 and D7.
- The benchmark report carries a third row from a run by the recorded procedure, with the shim's per-subcommand counts.

## Notes for the plan writer

- The signal entry's one install function already documents that a zero-pin rebuild with the watchdog on installs the entry; D4 adds a call site in the boot path after its `pinGeometryOptionsLocked` call, not a second writer.
- The poll timer can reuse the loop's `oneShot` and `NextWakeCadence`; the box comparison is against `lastApplied`, which the loop already tracks.
- `socketDirFromEnv` in `socketfile.go` is the one place reed spells tmux's socket directory rule; the kit holds its own 107-byte constant.
- The daemon's engine override belongs in `reedcli`, which already loads the worktree config in `enterSession`.
- The logger's package doc states "Debug never reaches the durable sink" in several places; each must follow D1.
- The measurement card is manual work in a scratch hub, as the earlier report records, never a test.
