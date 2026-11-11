# Cap the skill-load API-error hold and report idle gaps in shed status

A shuttle start whose skill-load turn ends on an API error such as `server_error` used to wait and retype with no bound.
In the reported run `apiErrors` rose 3, 4, 5 until the loop ended with no envelope, and hours of idle time around the stuck step showed in no status line.
This change bounds that hold and makes `lym loom status` show the idle stretches.

Closes #102

## What changes

- **Capped hold (`shuttleengine`).**
  The hold now ends the start as `api-error` at its `loadHoldMaxAPIErrors`-th API error (8), with no load typed after it.
  Its back-offs add up to about half an hour before the cap fires.
  `Result.APIError` carries the newest error text, and the start's error names the cap, so a halt reads as an API outage rather than only `loop-exited`.
  The parent's hold notice now states the cap.
  A run whose output files all exist still returns as past its gates, as on the other load-turn endings.
- **Idle gaps (`shedengine`, `shedverbs`).**
  Each history entry a producer call writes records the call's start in the new `started_at` field beside `at`.
  `lym shed status` reports `idle_gaps` in its envelope, and its human view prints one `idle` line per gap, oldest first.
  A gap is a stretch of five minutes or more (`idleGapFloor`) from one entry's end to the start of the call that wrote the next.
- **Docs.** `docs/overview.md` and the package docs of `shuttleengine`, `shedarbiter`, `sheddriver` and `shedverbs` describe both changes.

## Design decisions

- The cap counts API errors of one hold, so the cap and its back-offs bound the total wait.
  An outage past about half an hour is the parent's to judge, and the existing `api-error` hand-on carries it there.
- For a goto or skip entry, which no producer call writes, a gap ends at the entry's own time.
- Calls folded into one stuck entry count as one stretch, so the time between them is not reported.
- An entry with no `started_at` (a status file an older binary wrote) or with unparsable times ends no gap, so old runs read without error.

## To check by hand

- `startedat-pinned-only-on-done-path`: `StartedAt` is set on the done path in `shedengine/run.go` and on the reroute entries in `shedengine/reroute.go`.
  Check that the tests pin it on each of those paths and not only on the done path.
- No run exercised a real provider outage, so the 8-error cap has not been seen firing against a live API.
