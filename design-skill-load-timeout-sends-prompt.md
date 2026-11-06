# A skill load that times out still sends the prompt, and the session runs without its skills

`skill-load-timeout-sends-prompt` · Note · bug, shuttle

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

## Idea

Shuttle sends the prompt only after the load turn is verified.

- A load turn that timed out or ended on an API error is retried once the API answers, and the prompt waits for that, as a pause already holds a send.
- When skills are still missing after the retries, the seat stops and says so, instead of running without them.

A skill unknown to the session is a separate case, and whether it keeps today's skip is a call for the task.
