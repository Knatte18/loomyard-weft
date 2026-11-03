# An arbiter whose turn ends on an API error holds the stop silently until its deadline

`arbiter-api-error-stall` · Task · bug, arbiter, shuttle, claude · running · PR-Review

An arbiter session whose turn ends on an API error sits idle until its deadline, so the stop it holds reaches no one for half an hour.

## Seen

On the `cs` hub (2026-11-03), `report-significant-digits` blocked at Drift-Await; the tackler's second attempt escalated at 17:06:03 and the arbiter was spawned (`_lym/shed/report-significant-digits/arbiter/attempt-1/attempt.json`, deadline 17:36:03).

- Three seconds later the session's turn ended on `API Error: 500 Internal server error`: its `events.jsonl` holds `StopFailure` at 16:06:06Z with `"error":"server_error"`, then `idle_prompt` at 16:07:06Z.
- No verdict was written and nothing was posted to `cs:orch`: the hub's creel log holds no post about the stop between the block and 17:33.
- `cs:orch`, told of the stop by another session, repaired it and ran `lym loom resume` five times between 17:32:28 and 17:33:53; each was refused with `"kind":"driver_not_parked"`, "the run's Go driver still holds its driver lock and is settling the stop; run \"lym loom resume\" again in a few seconds", while the dead arbiter had held the stop for 26 minutes.
- At 17:34:41 a fresh arbiter was spawned in the same `attempt-1` (`go-settle.jsonl`: `"settler":"arbiter","action":"spawn"`); it found the repair and resumed the run in ten seconds.

## Expected

- lym reads a `StopFailure` hook event of a settler's session (arbiter, tackler, and any other role run under a deadline) as the end of that turn: it re-sends the turn after a short back-off a bounded number of times, then hands the stop on to the parent with the error named.
- A stop held by a settler whose session has failed is never silent: the parent orch gets a notice naming the stop and the failed settler.
