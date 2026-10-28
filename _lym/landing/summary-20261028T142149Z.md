# One notice per usage limit, and the orch is woken after the reset

When Claude's usage limit stopped a task's agent, batten told the orch the same api-error notice again and again until the limit reset, and the orch, stopped by the same limit, could act on none of them.
After the reset nothing woke the orch; the operator had to.

## What changes

**Shuttle.**
`TurnEndActivity` and `AgentActivity` carry a `UsageLimit` reading: whether the newest main-chain turn end is an API error the provider classifies as its usage limit, and when the limit resets.
`Runner.UsageLimit` gives the same reading for one turn end of a named run.
The Claude engine takes the limit from its `rate_limit` classification and the reset time from the message's `resets 2pm (Europe/Oslo)` clause, with optional minutes and an optional month and day, as the first such instant after the entry's timestamp in the named zone (`claudeengine/usagelimit.go`).
The zone database is embedded through `time/tzdata`, so the zone resolves on Windows too.

**Batten.**
A run standing on a usage limit gets a `usage-limit` notice, ahead of the api-error and quiet ones, naming the agent, the reset time in UTC or `reset time unknown`, and the limit's text.
Its episode key is stamped with the reset time and stays sent while any live run of the child reads usage-limited, however its activity moves; a later limit sends a new notice.
No api-error revive is sent for it.
The repeats came from that revive: each one typed into the limited session, which ended in the limit again, moved the activity and so opened a new api-error episode, and from the third in a row every one was noticed.

**Orch watcher.**
When the newest turn end an idle tick reads is the orch's own usage limit, the watcher records it in `State` (`UsageLimitAt`, `UsageLimitResetsAt`, `UsageLimitWakeAt`), holds the orch strand's creel deliveries and starts no cycle.
One minute after the reset time, or 30 minutes on when none is readable, it injects one line from the new `orch-template-usage-limit-wake` stencil, telling the orch to run `lym tender revive` and check each run, and repeats it every 30 minutes until the orch answers.
The first turn end that is not a usage limit clears the record and releases the hold, so the held notices are delivered.
A fresh launch clears the record.

## Design decisions

- The orch's own turn end is the reading the watcher acts on: the orch shares the account's limit, so no hub-wide limit record is needed between batten and the watcher.
- The wake line is the watcher's own inject, which its creel hold lets through, so the hold can stand until the orch answers.
- A wake that creel refuses stays due and is tried at the next tick; a wake line that fails to render waits for the next retry, with a Warn.
- One notice per task run and limit stands for the issue's "one per strand": a run's agents share it.
- `lym tender revive` is unchanged: it already skips a session Claude Code resumed on its own as busy.
- A limit hit during a cycle is not read until the next idle turn end; the cycle's own timeouts cover it.

## Tests

- `TestUsageLimitResetsAt` covers the reset clause: with and without minutes, a time already past, midnight, a date, a date in the next year, the older wording, an unknown zone, no clause, an hour out of range and a bad timestamp.
- `TestTurnEndActivity` gains a usage-limit row.
- `TestNotice_UsageLimitNotifiesOncePerLimitWithoutARevive` pins one notice and no revive across moving activity, and a second notice for a later limit, with and without a reset time.
- `TestWatcher_UsageLimitHoldsTheOrchAndWakesItAfterTheReset` pins the hold, no cycle over the threshold, the first wake, the retry and the release, with and without a reset time.
- `TestRenderPointers_*` gain the wake line's rows.

Closes #894.
