# A skill load stopped by an API error counts as skipped, and the prompt is sent into the outage

`skill-load-timeout-sends-prompt` · Task · bug, shuttle · draft

## Seen

During the network drop of 2026-11-06, about 13:39 to 13:55, the Code-Bouncer judge of `assist-fan-seat-work-alone` launched with `scribe:prose`.
Its load turn hit an API error, `shuttle: skill skipped ... cause=timeout` was logged at 13:42:21, and shuttle typed the prompt line into the session anyway, which hit the same API error.
The session then stood idle until the operator typed `fortsett` by hand, and it loaded the skill and carried on only because the load message was still in its context.

## Cause

`internal/shuttleengine/run.go`: `settleLoadTurn` returns no error when a load turn times out, and `loadSkillsThenPrompt` goes on to the prompt.
That is the written design, in `internal/shuttleengine/doc.go`: "a turn that never ends skips its skills at the timeout with no retry", and "a skipped skill never fails or hangs it".
An unverified turn goes on the same way.
A skipped skill reaches only the log, so the session's agent, the run and the orch never learn it.
The default timeout is 2 minutes (`defaultSkillLoadTimeout`), shorter than a usual network drop.

## Decision (operator, 2026-11-06)

Going on past a failed load is right only when the skill is not installed or is missing: that keeps today's skip with its warning, and the session never stops for it.
An API error is a different case: it says nothing about the skill, so the load has not failed.

## Wanted

- A load turn that ended on or timed out through an API error is not a skipped skill: shuttle waits until the API answers, runs the load turn again, and sends the prompt only after it, as a pause already holds a send.
- The prompt line is not typed into a session whose API is failing.
