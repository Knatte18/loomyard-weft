# shuttle: hold a skill-load turn the API failed, don't type the prompt

A start whose skill-load turn ended or timed out through an API error no longer counts the skills as skipped, and no longer types the prompt line into the failing session.
During the network drop of 2026-11-06 a judge launched with `scribe:prose`: its load turn hit an API error, shuttle logged `skill skipped ... cause=timeout`, typed the prompt into the same failing API, and the session stood idle until the operator typed `fortsett`.
An API error says nothing about the skill, so the load has not failed.
A skill that is not installed or is missing keeps today's skip with its warning.

## Behavior

The load turn now lives in `internal/shuttleengine/loadturn.go`.

- A load turn that ends on an API error, by the engine's signal or by the transcript's marker on a turn end, skips nothing and spends no retry.
  Shuttle waits a back-off from `apiErrorRetryBackoff` doubling to `loadHoldBackoffCeiling` (5 minutes), types the same load message through the verified send path, and repeats until a load turn ends.
  The typed load line is the only probe of the API a session has, and the prompt follows only after a load turn ends.
- The send path is the pause-aware one, so a pause holds the re-typed load too.
  A busy session moves the line to the next back-off.
  A turn start read during the back-off takes the place of the typed line.
  A pane that dies ends the hold as a died startup.
- A turn start read with no turn end after it is awaited until its end, the pane's death or the run's deadline, since a provider retries a failing API inside a turn for minutes.
  `SkillLoadTimeout` now ends only a turn no turn start was read for.
  A turn still open at the run's deadline fails the start as `OutcomeTimeout` with no prompt typed.
  An engine with `SessionProber` that reads the open turn as interrupted skips its skills at once with no retry, since an interrupt writes no turn end.
- The hold has no bound of its own and moves the run's deadline later by its length, as a pause does.
  At the second API error it sends the run's parent one notifier line, skipped for an interactive run.
- A turn end the engine marks `ModelRejected` ends the start at once as `OutcomeAPIError` with `Result.APIError` naming the error, since no further turn changes the model.
- Bound: an attached run delivers a pending prompt without judging the API, so a driver that died during a hold has its prompt typed by the attach.

`doc.go`, `engine.go`, `spec.go`, `run.go` comments, `PATTERN-reed-panes.md` (new rule "An API error holds the prompt") and `PATTERN-daemon-wakeups.md` (the hold listed as an unbounded wait, with `loadturn.go` added to `unboundedWaitFiles`) describe the new behavior.

## Design decisions

- The typed load line is the probe rather than a separate API check, because a session offers no other way to ask whether its API answers.
- The hold is unbounded by design: a bounded hold would put back the skip-and-type-anyway behavior for a long outage.
  The run's deadline moves later by the hold so an outage does not time the run out.
- An open turn is awaited past `SkillLoadTimeout`, since the timeout was shorter than a usual network drop and a provider keeps retrying inside the turn.

## To check by hand

Nothing was run by hand, and no regression check ran for this task.

- `hold-behaviors-lack-pinning-rows`: confirm the rows of `TestStartup_LoadTurnAPIError` in `internal/shuttleengine/startup_test.go` pin every hold behavior above (back-off doubling and ceiling, busy session, turn start during the back-off, dying pane, failed send, notice once and none when interactive, notifier failure, model rejected, open turn at the deadline, interrupted turn).
- `open-load-turn-skips-and-types-prompt`: confirm in `loadSkillTurn` (`internal/shuttleengine/loadturn.go`) that an open load turn past the timeout neither skips its skills nor lets the prompt be typed, and that `TestStartup_LoadTurnAPIError` pins it.
