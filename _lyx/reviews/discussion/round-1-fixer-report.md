# Fixer report — discussion round 1

Every finding was validated against the code and fixed; none is disputed or deferred.

## Fixed

- F1 (BLOCKING, D2): confirmed against `internal/reedengine/spawnwatchdog.go` (nil `Env`, so the environment is inherited) and `internal/logger/logger.go` (`configureFromEnv` calls `SetOutput` for `LYX_LOG_FILE`).
  D2 now has `SpawnWatchdog` pass the daemon its environment minus `LYX_LOG_FILE`, keeps `LYX_LOG_LEVEL`, and states a bound: a foreground run's stderr half goes where that run's operator pointed it, and a detached daemon's reaches nothing, its Debug trace only the capped durable sink.
  Scope In names the `SpawnWatchdog` change; a new acceptance criterion pins the child environment.
  The support log records the rejected terminal-probe or spawn-flag alternative.
- F2 (MEDIUM, D4): confirmed `pinGeometryOptionsLocked` runs from `lifecycle.go` and `attach.go`.
  D4 now places the zero-pin rebuild in the boot path after that call, never inside it, and says why; the attach pre-flight keeps its order.
  A criterion pins that a degrading attach with the watchdog on leaves the array untouched; the plan-writer note names the new call site.
- F3 (MEDIUM, D7): confirmed `template_posix.yaml` has no key for any operator tmux setting.
  D7 gains a bound: `-f /dev/null` skips the operator's whole `~/.tmux.conf` on reed's per-hub servers only, reed sets what it needs itself, no `reed.yaml` key carries an operator setting, and hand-sourcing into the running server is the only route, lasting until it exits.
  D7 also states that strand-pane shells run rc files and no longer the login profile.
  The Open risks line is corrected to match.
- F4 (LOW, D3): the bound now says what happens: the daemon uses the spawning worktree's `tmux` value and ignores every other worktree's, a difference is accepted silently, and why a comparison warning would fire on healthy hubs.
  The rejected warning goes into the support log.
- F5 (LOW, D5): confirmed `watchloop.go` parks on `ctx.Done()` under `watchdog: off` and stays in poll mode after a failed promotion.
  The bound is now the 60-second ceiling on both platforms, naming the POSIX cases that stay in poll mode and the parked `watchdog: off` loop; the Open risks line follows.
- F6 (LOW, D4): reworded to POSIX `watchdog: off` keeping its unset and Windows issuing no hook call, matching `windowsize.go` and the criterion.
- F7 (LOW, D6): the limit is now per-OS, 107 bytes on Linux and 103 elsewhere on POSIX; the criterion follows, and the support log records the rejected single limit.
- F8 (LOW, D7): the attach pre-flight resolves the shell again with `exec.LookPath`, and on failure warns and skips the two shell pins, as its other pins degrade.
  A criterion pins that degrade, and the boot refusal criterion now carries D7's POSIX qualifier.
- F9 (LOW, D8): the attribution is stated as a working hypothesis that the re-run's per-subcommand counts and children CPU confirm or refute; on refutation, the report records what the counts show and changes no code.
  The support log's interview item 5 and Q10 follow.
- F10 (LOW, D6, D7): D7 now states that the shell options are session-scoped and never `-g`, with the reason, before the narrative leaves; the alternative-arguing sentences in D6 and D7 are dropped, since the support log already holds both.
  A criterion pins the never-`-g` scope.
- F11 (NIT): confirmed `contracts/stencils/landing/landing-template-describe.md` writes the `Closes #N` lines.
  The `Closes #495` constraint is dropped, and the criterion is cut to the PATTERN background-file half; the refusal constraint stays, as the review says.

## Disputed

None.

## Deferred

None.
