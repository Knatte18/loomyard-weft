```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff for code correctness, as lens code-correctness.

**Per-function list (case run → result)**

- `usageLimitResetsAt`, `resets 2pm (Europe/Oslo)` at 11:03Z (12:03 Oslo): 14:00 Oslo, after the timestamp, so 13:00Z. Correct.
- `usageLimitResetsAt`, 12am and 12pm: `hour%12` then +12 for pm gives 0 and 12. Correct.
- `usageLimitResetsAt`, 13pm, 0, or a minute above 59: zero. Correct.
- `usageLimitResetsAt`, a time already past: rolls to the next day. Correct, DST included.
- `usageLimitResetsAt`, a month and day already past: rolls to the next year. Correct.
- `usageLimitResetsAt`, `Feb 31`: `time.Date` normalises to Mar 3 instead of reading as no time. Wrong, but harmless (see F2).
- `usageLimitResetsAt`, unknown zone, no clause, or unparseable timestamp: zero. Correct.
- `newestTurnEndError`, a non-API-error turn end gives an empty reading, and a `rate_limit` API error gives `Limited` with the parsed time. Correct.
- `readRunActivity`, a turn end before the prompt offset: `UsageLimit` is not copied. Correct.
- `Runner.UsageLimit`, an engine that is not an `ActivityReader` reads no limit; `FindRun` and `engineFor` errors propagate. Correct.
- `recordUsageLimit`, first limited reading with a future reset: sets `UsageLimitAt`, and the wake is reset plus 1 minute. Correct.
- `recordUsageLimit`, zero reset time: wake at now plus 30 minutes. Correct.
- `recordUsageLimit`, an already-limited record re-read: keeps `UsageLimitAt` and moves the wake. Correct.
- `recordUsageLimit`, a non-limited reading: clears the record and reports `ended` only if a limit stood. Correct.
- `recordUsageLimit`, reset time already past when the limit is first read, such as after the watcher was down: wake is now plus 30 minutes. Wrong against D5 (see F1).
- `tick`: the record is persisted, because `readingChanged` is always true when `lastTurnEnd` is set. Correct.
- `tick`, release on `limitEnded`: a failed release goes to `creelFailed`, and the hold lapses within `cycleHoldDuration`. Correct.
- `tick`, a `UsageLimit` error returns before the cursor advances, so the tick retries. Correct.
- `tickUsageLimit`, hold refused: returns with nothing sent. Correct.
- `tickUsageLimit`, wake not yet due: no send. Correct.
- `tickUsageLimit`, due: injects once, moves the wake 30 minutes on, saves. Correct.
- `tickUsageLimit`, render failure: defers 30 minutes. Correct.
- `tickUsageLimit`, inject failure: `creelFailed` leaves the wake due, so it retries each tick. Correct.
- `tickUsageLimit`, inject with ttl 0: see F3.
- `ResetForFreshLaunch`: clears the limit fields. Correct.
- `keepEpisode`, `usage-limit` key: kept while activity is unknown or any run is limited, dropped when none is. Correct.
- `judgeAgents`, usage-limit finding: comes ahead of api-error, so no api-error revive for it. Correct.
- `resetStamp`, zero reset time: stamp 0 is a stable key per limit. Correct.
- `RenderUsageLimitWake`, zero and non-zero reset: both render to one line. Correct.

**Findings**

- **F1 — LOW, design.** A limit read after its reset time has passed waits 30 minutes instead of waking at once.
  - `internal/orchengine/usagelimit.go:33-36`: `wake.After(now)` is false for a past reset, so the wake falls through to `now + usageLimitRetry`.
  - D5 says reset plus one minute, and a past time would fire immediately.
  - It matters when the watcher was down at the reset or the turn end is read late.
- **F2 — LOW, design.** A month-and-day clause that is not a real date, such as `Feb 31`, normalises to another date instead of reading as no time.
  - `internal/shuttleengine/claudeengine/usagelimit.go:54-61`: `time.Date` normalises `Feb 31` or day 0 without error.
  - D2 says an unreadable clause reads as no reset time.
- **F3 — MEDIUM, design.** The wake line is queued with ttl 0.
  - `internal/orchengine/usagelimit.go:64`: a zero ttl takes creel's default ten minutes (`internal/creelengine/verbs.go:625`).
  - The wake is retried every 30 minutes, so an undelivered line expires after ten and no stale line pile-up is guaranteed.
  - The code reads as "no expiry" to a reader. The behaviour is acceptable, but the ttl should be `stepInjectTTL` like the other injects. No wrong result follows from it.

**Tool calls**
1. Read `internal/orchengine/watcher.go` 425-530 for `renewHold`, `releaseHold`, `creelFailed` and `orchName`, which the manifest lacked.
2. Grepped `Inject(` and `Hold(` call sites to see how the hold interacts with the watcher's own inject.
3. Grepped ttl handling in `internal/creelengine/verbs.go` to learn what a zero ttl means.

I did not check whether the watcher's own inject passes the watcher's own hold, since I had no creel delivery-path read. The decision record assumes it does.
