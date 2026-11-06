# Shuttle: hold the prompt while the API fails a skill-load turn

A skill-load turn that an API error stops no longer counts as a skipped skill, and the prompt is no longer typed into the outage.
Shuttle waits, runs the load again, and types the prompt only after a load turn has ended.
A skill that is not installed, or is still missing after the retry, keeps today's skip and warning.

## Why

During the network drop of 2026-11-06, a Code-Bouncer judge's load turn started at 12:40:20 UTC.
Shuttle's 2-minute timeout fired at 12:42:21 and logged the skill as skipped, while Claude Code was still retrying inside the turn.
The turn ended on an API error at 12:43:21, the prompt went in less than a second later and hit the same error, and the session stood idle until the operator typed by hand.
Another session in the same outage took about seven minutes to end its turn on the error, so a load timeout alone cannot tell a failing API from a slow turn.

## What changes

`loadSkillTurn` moves from `run.go` to the new `internal/shuttleengine/loadturn.go`.

- **API error at a turn end.** A turn counts as failed by the API on the `SessionSignalAPIErrorTurnEnd` signal, or on a turn end the transcript marks as an API error.
  Shuttle sleeps a back-off of 30 seconds, doubling to five minutes, then types the same load message through the verified send, so a pause holds it too.
  This repeats until a load turn ends, and spends neither the one retry for missing skills nor a skip.
- **Open turn at the timeout.** For an engine that reads turn starts, a turn still open at the timeout is awaited up to `loadTurnOpenBound`, 15 minutes from its start.
  That is what catches the case above.
  An engine with no signal reading, and a turn that never started, still skip at the timeout.
- **A turn another party starts.** A turn start read during the back-off replaces the typed line, so a dobby revive or an operator's line carries the load on, and its turn end is classified as the load's.
  A busy session moves the line to the next back-off.
- **Deadline.** The run's deadline moves later by the length of the hold, as it does for a pause.

## Design decisions

- **The typed load line is the probe.** A session has no other way to ask whether its API answers, and the decision record asks for the load to run again anyway.
  Each failed attempt adds a line and an error entry to the session's context, which is why the back-off has a five-minute ceiling rather than a shorter one.
- **The hold has no bound of its own**, matching how a pause holds a send.
  An error that will not pass, such as a failed login, holds the start until it is stopped.
  So the second API error sends the parent one notifier line, skipped for an interactive run.
  The decision record did not ask for this notice; it is there so a lasting hold shows somewhere other than the log.
- **A rejected model ends the start at once.** No further turn changes the model, so a turn end marked `ModelRejected` ends the start `api-error`, with `Result.APIError` naming the error and nothing typed.
  It goes through the `OutcomeDone` check first, and is not relaunched by `start_retries`.
  The decision record does not name this case; holding it would hang forever.
- **A hung turn now takes longer to fail.** A load turn that stays open with no API error used to reach the prompt send after the timeout; it now waits out `loadTurnOpenBound` first.
  An operator's interrupt during a load turn writes no turn end, so it waits the same bound unless the operator types.

## Known limit

`Run.DeliverPendingPrompt` does not judge the API, so a driver that dies during a hold has its prompt typed by the attach, even if the API is still failing.
The second wanted point therefore holds for the start path only.
The bound is stated in the package doc and in `PATTERN-reed-panes`.

## Docs and tests

- The package doc's skill-loading section, `Spec.SkillLoadTimeout` and `OutcomeAPIError` describe the new behavior.
- `PATTERN-reed-panes` gains the rule, and `PATTERN-daemon-wakeups` and the scan in `cmd/lym/daemonwakeups_test.go` list the new wait.
- `TestStartup_LoadTurnAPIError` covers both API-error forms, a timeout followed by an API error, the doubling back-off, the single notice, the spared retry, the moved deadline, a pane dying during the hold, and an open turn reaching its bound.
  `TestRun_LoadTurnModelRejectedEndsTheStart` covers the rejected model.
