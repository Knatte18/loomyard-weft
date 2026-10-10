# Card 3 — batten-awake-idle-clock

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.judgeAgents`
- `internal/battenshed/doc.go`
- `internal/battenshed#noticeHarness`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#TestNotice_QuietOnlyForRunningChildWithLiveDriver`
- `internal/battenshed#TestNotice_APIErrorStall`

**Create:**
- `plan:internal/battenshed#sleepGapTolerance` -> `const sleepGapTolerance = time.Minute`
- `plan:internal/battenshed#sleepGap` -> `func sleepGap(prevWallNanos, wallNanos int64, prevAwake, awake time.Duration) time.Duration`
- `plan:internal/battenshed#childWait.observeClocks` -> `func (w *childWait) observeClocks(wallNanos int64, awake time.Duration, stamp activityStamp)`
- `plan:internal/battenshed#childWait.awakeSince` -> `func (w *childWait) awakeSince(now, at time.Time) time.Duration`
- `plan:internal/battenshed#noticeHarness.suspend` -> `func (h *noticeHarness) suspend(d time.Duration)`
- `plan:internal/battenshed#TestSleepGap` -> `func TestSleepGap(t *testing.T)`

**Uses:**
- `internal/battenshed#innerRunProducer.noticeCondition`
- `internal/battenshed#innerRunProducer.agentActivity`
- `internal/battenshed#activityStamp`
- `internal/battenshed#noticeAPIErrorIdle`

**Intent:** D3 of the decision record: the quiet and api-error idle clocks measure awake time, dropping the time the machine spent suspended.

Seam. `InnerRunDeps` gains `Awake func() time.Duration` beside `Now`: a clock that stops while the machine sleeps, read as a duration from any fixed origin.
`NewInnerRun` resolves a nil `Awake` to Go's monotonic reading, a closure returning `time.Since` of an origin taken at construction, once, like `Now` and `Sleep`; the doc comment names the platform bound of D3 (Linux and macOS stop the monotonic clock during suspend, Windows keeps today's behaviour).

Helpers in `notice.go`.
`sleepGap` takes the two consecutive wall readings as `UnixNano` values, so its inputs carry no monotonic part by construction, and the two awake readings; it returns the excess of the wall delta over the awake delta when that excess exceeds `sleepGapTolerance`, else zero, and reads a negative delta as zero.
`childWait` gains the previous wall and awake readings with a flag for their presence, the accumulated gap, and the activity stamp the accumulation belongs to.
`observeClocks` is the one updater: with no previous readings it records the wall and awake readings and, when known, the stamp, and returns, so only a stamp that moves after a recorded one drops the accumulation; otherwise it computes the gap since them, records the new readings, and then either drops the accumulation without adding the gap when the stamp is known and differs from the recorded one, recording the new stamp, or adds the gap when the stamp is unknown or unchanged (the carry-over finding d3-stamp-change-drop-versus-add-order is settled as drop, never add, on a changed stamp).
`awakeSince` is the one subtraction: `now.Sub(at)` minus the accumulation, the delta every idle check below uses.

`judgeAgents` reads `Now` and `Awake` on every check it runs and calls `observeClocks` before its early return on an unreadable activity, with an unknown stamp in that case, so the gap is kept and added; in the no-live-run branch the stamp is the fallback's (the later of the newest history entry and the episode's since) and in the live-run branch the newest `LastActivity`, computed before the call.
Its three comparisons then go through `awakeSince`: the fallback quiet check against its stamp, the api-error check against each erroring run's own `LastActivity`, and the quiet check against the newest activity; both notice texts report the `awakeSince` duration.
`noticeKey` and `keepEpisode` are untouched: the stamps are unchanged, only the deltas shrink.
No new wake is added (PATTERN-daemon-wakeups): both clocks are read on the checks the wait already makes.

Docs: `doc.go`'s quiet-notice paragraph gains the rule that both idle clocks measure awake time, that suspended time observed between two checks is subtracted until the newest activity moves, that a sleep under a minute still counts as idle, and that the subtraction can delay a notice and never raise one earlier; `InnerRunDeps.Awake`'s field comment carries the seam's contract.

Tests, all tier 1 with the fake clock, no waiting (PATTERN-test-speed).
`noticeHarness` gains an `asleep` duration and wires `Awake` in `producer` as the fake clock's elapsed time since the harness started minus `asleep`; `suspend` advances the fake clock by `d` and adds `d` to `asleep`, so wall time moves and awake time does not.
`TestNotice_QuietOnlyForRunningChildWithLiveDriver` gains subtests: a sleep gap of hours after the newest activity raises no quiet notice until the awake idle time reaches the window, and the notice text reports the awake duration (AC7); a check that observes both a gap and a moved newest activity drops the gap, so the quiet notice fires at the window measured from the new activity (AC9 and its row); the no-live-run fallback drops the gap when its stamp moves (AC8b); an unreadable activity keeps the accumulation and adds its gap, which a later readable check subtracts.
The existing subtests stay unchanged and pass with wall and awake advancing together (AC8).
`TestNotice_APIErrorStall` gains a step: a gap of hours after the stall raises no notice until the awake idle time reaches `noticeAPIErrorIdle` (AC8b).
`TestSleepGap` is a table test over the helper's `UnixNano` inputs (AC8a): hours of wall against seconds of awake yields the excess, an excess at or under the tolerance yields zero, equal deltas yield zero, a wall delta behind the awake delta yields zero.
The package's top-level test count stays under its row in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** One new optional field on `InnerRunDeps`, defaulted in `NewInnerRun`; three idle comparisons in `judgeAgents` change their delta; no caller outside `internal/battenshed` changes, since `battencli` wiring leaves the field nil.
