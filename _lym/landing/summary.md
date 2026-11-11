# Cap the skill-load API-error hold; show idle gaps in loom status

A skill-load turn that the provider's API ended on an error, such as a `server_error`, made shuttle wait a back-off and type the load again, with no bound.
In the reported run the count rose to five before the loop ended with no envelope, and hours of idle time around the stuck step showed nowhere.
After this change the hold ends at a cap, and the step fails with an error that names the cap and reads as an API outage.
`lym loom status` also shows each stretch of the run's history in which no step ran.

Closes #102

## What changes

- `internal/shuttleengine/loadturn.go`: the hold's `loadHoldMaxAPIErrors`-th (8th) API-error turn end ends the load with a new `loadAPIErrorCapError`.
  Its text reads `the provider's API failed 8 skill-load turns, the cap on one start's hold of 8, so the start ends as an API outage: <error>`.
  `abandonLoad` ends such a start `OutcomeAPIError`, as it does a rejected model, with `Result.APIError` naming the error.
  The start's error wraps `ErrNotStarted` as before, so the cap text reaches the step's failure and the run's status `error`.
  The parent's hold notice now says at how many API errors the start ends.
- `internal/shedengine`: each history entry a producer call writes records the call's start in a new `started_at`, beside `at`.
  This covers the reroute entries too.
  A fold keeps the first call's `started_at`, as it keeps its `at`.
  A status file without the field still decodes.
- `internal/shedverbs`: the status envelope carries `idle_gaps`, and the human view prints one `idle` line per gap.
  A gap is a stretch of at least `idleGapFloor` (five minutes) from one entry's end (`last_at` or `at`) to the next call's `started_at`.
  For a goto or skip entry, which no call writes, the gap ends at that entry's own `at`.
  The human line reads `idle 3h0m0s before Plan, after Plan stuck`.
- Doc comments in `shuttleengine` and `shedverbs`, and the status section of `docs/overview.md`, follow both changes.

## Design decisions

- The cap counts every API-error turn end of the hold, not only `server_error`.
  The session signal carries no error kind, and the load hold already treats every API error alike.
  A usage limit that outlasts the hold therefore also ends the start.
- Eight errors keep the back-offs (30 s doubling to the 5 min ceiling) at about half an hour, besides the time the provider spends retrying inside each turn.
  Past that, the outage is the parent's to judge.
  The failure is classified `agent-start` transient, so one re-step can still clear it.
- The idle time is derived from the history instead of being stored.
  An idle stretch is only known once the next step starts, and `started_at` is enough to compute it at read time.
  Entries an older binary wrote have no `started_at` and end no gap, so a gap is never overstated by a step's own run time.
- A gap counts as idle whatever came before it: a stuck verdict, an awaiting hand-off or a step that died without recording a verdict.
  In the reported case, the step that hung left no entry at all, so the gap shows before the entry its resumed call wrote.

## Tests

- `TestStartup_LoadTurnAPIError` gains a row in which every load turn fails.
  The start ends `api-error` after the derived back-offs, with the load typed seven times, one parent notice and the cap named in the error.
- `TestStep_DoneWithNonEmptyOnDone` checks that `started_at` is set and no later than `at`.
- `TestIdleGapsOf` covers a gap at the floor, one under it, a folded entry, a goto, an entry with no start time and an unparseable time.
- `TestRenderStatusHuman` gains an `idle` line row, and `TestStatusCmd_FoundEnvelope_GenericCore` pins `idle_gaps` as an empty list for entries with no times.
