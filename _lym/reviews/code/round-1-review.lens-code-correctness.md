```yaml
lens: code-correctness
outcome: reported
```

**Task:** the code-correctness lens. I traced each changed function and error path with a concrete input. Everything below is correct unless marked.

**List (function or branch, case run, result)**

- `waitReminders` guard: nil `Post` or nil `ReadChild` returns at once. Correct.
- `waitReminders` throttle: a first call with a zero `lastWaitCheck` runs, and a call 10s later returns. A call at least 60s later runs; the loop's `now` is taken at tick start, so the gap is ≥60s. Correct.
- `waitReminders` state unreadable: it logs `|state` and returns. `lastWaitCheck` is already advanced, so the retry comes in a minute. Correct.
- `readWaits`, run not running or `CancelMarker` set: skipped. It is in neither `waits` nor `unread`, so `recordWaits` clears a recorded wait. This matches "clears when the child stops running". Correct.
- `readWaits`, a read error: the slug lands in `unread`, so the record stands and `lastReminders` is kept. Correct.
- `readWait`, `ReadChild` found=false, state not running, or a row outside `Rows`: returns zero. Correct. `Pause` is only read for a listed row.
- `readWait`, `Pause.Covered`: returns zero, which clears the wait. A later uncovered read starts a fresh `Since=now`. Correct.
- `readWait`, row changed: a new wait with `Since=now`. Correct.
- `readWait`, `LatestResume` returns zero: `After` is false, nothing moves. Correct.
- `readWait`, `resumed` later than `Since`: `Since=resumed`. A resume before the first read is not later than `now`, so `Since` stays `now`. This gives the later of the two, as D8 says. Correct.
- `readWait`, `LatestResume` errors: returns the error and the slug is unread. Correct.
- `recordWaits` changes set: the `!=` on `OperatorWaitState` holds a `time.Time`, so it compares loc pointers and the monotonic reading. The unchanged case copies `recorded`, so it is struct-equal and no rewrite happens. A new `Since=now` differs from zero and is written once. Correct.
- `recordWaits` unrecorded slug: the `for slug := range waits` loop adds a new `RunState`. `UpdateState` creates the map and the entry. This is the same `s.Runs[slug]` pattern `runnotice.go:409` and `owned.go:20` use. Correct.
- `recordWaits` `lastReminders` pruning: `maps.DeleteFunc` on a nil map is safe. A slug is dropped only when it is neither waiting nor unread. Correct.
- `recordWaits` write failure: Warn only. The reminders still go out from the in-memory `waits`. Correct.
- `remindOfWaits`, `Away` error: logs `|presence` and returns without posting. Correct.
- `remindOfWaits`, not away: `clear(lastReminders)`. A return to slack gives `reminded=false`, and a wait past the interval is reminded at once. Correct.
- `remindOfWaits`, away with empty `waits`: returns before reading the interval. Correct.
- `remindOfWaits`, `Interval` error: no post. `LoadConfig` returns an error for a value under 1, so a zero or negative `remind_after_min` posts nothing and logs once per episode. Correct.
- `remindOfWaits`, cadence: a post at T sets `last=T`, and the next is allowed once `now-last ≥ interval`. The check spacing is ≥60s, so a reminder can drift up to one check later per interval. D8 says "each further interval", so this is acceptable.
- `remindOfWaits`, post failure: Warn, `last` not advanced, retried at the next check. A failed second reminder still has `now-last ≥ interval`, so it retries too. Correct.
- `remindOfWaits` line text: `int(waited.Minutes())` truncates. Correct.
- `waitReminderDeps.LatestResume`: `Read` then `LatestResumed`, returning the zero `ResumedAt` when none is found. The integration test covers the missing record. Correct.
- `waitReminderDeps.Away`: `presence == slack` together with `err`; the caller discards the bool when `err` is set. Correct.
- `waitReminderDeps.Interval`: `Duration(min)*Minute` with `err`. On an error `Config{}` gives 0 and the error is handled by the caller. Correct.
- `waitReminderDeps.Post`: the closure uses the outer `slug`, which is `geom.Slug`. This is the same scoped-program identity the existing notice uses. Correct.
- `Record.LatestResumed`: a hub-scope entry covers any slug, and a slug-scope entry covers only its own. An empty slug matches only hub entries. A live entry is not in `Resumed`, so a pause still open yields not-found. A tie keeps the first entry. Correct.
- `slackengine` config and template: the only changes are comment and wording edits. `LoadConfig` already refuses values under 1. Correct.
- `loop.go` wiring: `waitReminders` runs on every tick in the no-anchor-change branch. A probe-tick gate is not needed, because it throttles itself. Correct.

**Findings**

None at BLOCKING or MEDIUM from this lens.

**LOW, design, `internal/dobbyengine/waitreminder.go:118-122`**
- `recordWaits` writes `changes[slug]` computed from a snapshot read earlier, and `UpdateState` recreates `s.Runs[slug]` for a slug that a concurrent writer may have pruned meanwhile.
- A clear for a pruned slug leaves an empty `RunState`.
- This is harmless: the entry is empty and the same pattern exists elsewhere.
- Evidence: `waitreminder.go:143-145`, `passes.go:313`.

**LOW, design, `internal/dobbyengine/waitreminder.go:66-79`**
- A failure episode keyed by `run.Slug` in `waitFailures` is never recovered once the run stops being running or leaves `d.runs`, because `Recover` is only called on a successful read.
- A later failure for that slug would log at Debug instead of Warn.
- This is cosmetic.

**Tool calls**
1. `grep` for `.Runs[` and `range .Runs` in `internal/dobbyengine`: to see whether empty `RunState` entries from `recordWaits` matter to other readers. My context lacked those sites.
