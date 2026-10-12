# Card 4 — startup-probe-backoff-and-dismissal-gap

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsCounted` -> `func playInputsCounted(reed ReedOps, guid string, inputs []PaneInput) (sent int, err error)`
- `plan:internal/shuttleengine#TestStartup_DismissalGap` -> `func TestStartup_DismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#partialKeyReed` -> `type partialKeyReed struct`
- `plan:internal/shuttleengine#partialKeyReed.SendKey` -> `func (r *partialKeyReed) SendKey(guid, key string) error`
- `plan:internal/shuttleengine#twoKeyGateEngine` -> `type twoKeyGateEngine struct`
- `plan:internal/shuttleengine#twoKeyGateEngine.TrustDismissSequence` -> `func (e *twoKeyGateEngine) TrustDismissSequence(capture string) []PaneInput`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#newFakeClock`
- `pattern/PATTERN-daemon-wakeups.md`

**Intent:** Decisions D5 and D6 of the decision record, with the parent's later note that `awaitStartup` backs off under `PATTERN-daemon-wakeups` instead of amending it.
The two land together because the throttle is observable only once probes come closer than the gap: today both `awaitStartup` and `Wait` probe exactly one gap apart.

`dismissalGap` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls` floored to 1, the multiple `Wait` already floors the same way: 10 s under the shipped template.
`Wait` keeps its own integer flooring and tick arithmetic and is not changed.

`awaitStartup` probes at the daemon-wakeups backoff: its sleep starts at the base `pollInterval(cfg)`, doubles after every probe that left `run.lastStartupCapture` unchanged (a failed capture leaves it unchanged), is capped at `dismissalGap(cfg)`, and returns to the base after a probe that changed it.
Each probe still runs `checkLivenessTick`, one reed status and, before ready, one pane capture, and the clock, deadline and tick-cap exits are unchanged.
Its tick cap is `startupTickCap(window, pollInterval(cfg))`, the base interval, an upper bound the backoff only lowers; `startupTickCap`'s doc comment, which says it is given the probe interval and not the raw poll interval, is rewritten to say it is given the base poll interval.
The probe-cadence paragraph of `awaitStartup`'s doc comment is rewritten to state the backoff, the ceiling and that gate dismissals are spaced by `checkLivenessTick`'s throttle rather than by the probe cadence, one sentence per line.
No sub-second literal is added, and nothing is added to the daemon-wakeups guard's declared list, which already names `wait.go`.

`Run` gains an unexported field `lastDismissalAt time.Time`, the run clock's time of the last startup-gate play that sent at least one key, zero when none has.
`checkLivenessTick`'s `StartupTrustPrompt` case plays the trust-dismiss sequence only when `lastDismissalAt` is zero or the clock's now is at or past `lastDismissalAt` plus `dismissalGap(cfg)`.
It plays through `playInputsCounted`; a play that sent at least one key, failed partway or not, sets `lastDismissalAt` to now, while an empty sequence or a play whose first key failed leaves the field as it was, so the next probe is free to dismiss.
A probe inside the gap that still sees a gate plays nothing, asks the engine for no sequence, and classifies the startup window as before.
The Warn and Info log lines stay as they are.
What the throttle lets through and its bound: it removes key presses only, never adds one, and the startup deadline and tick cap still end a gate that never clears; `Wait`'s own startup probe runs exactly one gap apart, so under the at-or-past rule it is never held.

`playInputsCounted` is today's `playInputs` loop returning how many inputs were sent before the first failure beside the error; `playInputs` becomes the wrapper that discards the count, so its callers are unchanged.

`internal/shuttleengine/template.yaml`'s comment on `liveness_every_n_polls` adds that the value times `poll_interval_ms` also spaces startup-gate dismissals.
`internal/shuttleengine/doc.go`'s startup-probe paragraph adds that the probe runs every poll interval, backing off while the pane does not change, and that a startup gate is dismissed at most once per that gap.

Tests, all on the fake clock, none waiting out an interval.
`TestStartup_ReadyProbeTiming`'s "probe cadence matches Wait" row is replaced by three rows under `PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60`: pending twice then ready with one unchanging capture elapses 3 s (sleeps of 1 and 2 s); pending twice then ready with a capture that differs on every probe elapses 2 s; pending six times then ready with one unchanging capture elapses 35 s (sleeps 1, 2, 4, 8, 10, 10), pinning the ceiling.
`TestStartup_TickCap` computes its expected tick count from `startupTickCap` with `pollInterval(cfg)` instead of the multiplied interval.
`TestStartup_DismissalGap`, a new table-driven test under `PollIntervalMS: 1000, LivenessEveryNPolls: 10`, scripts a `CaptureQueue` that differs on every probe so probes stay one second apart, and pins: a gate on eleven consecutive probes then ready sends two keys, at 0 s and 10 s, and hands the engine two captures; a gate whose first key fails (`SendKeyErr` set) on three consecutive probes then ready attempts a key on every probe; a two-key sequence from `twoKeyGateEngine` whose second key fails through `partialKeyReed` on eleven consecutive probes then ready sends four keys, two per play at 0 s and 10 s; and a gate that never clears under `StartupTimeoutS: 12` and `LivenessEveryNPolls: 5` is dismissed at 0, 5 and 10 s and ends died wrapping `ErrNotStarted`.
`partialKeyReed` embeds `*fakeReed` and fails the SendKey call whose ordinal its field names; `twoKeyGateEngine` embeds `*fakeEngine` and returns a Down key and an Enter key, the shapes `flakyStatusReed` and `undismissableEngine` already take.
The package's top-level test count reaches its row in `cmd/lyx/testdata/test-budget.yaml` without passing it.

**ImpactSummary:** awaitStartup and checkLivenessTick change cadence and key replay only inside shuttleengine; Wait's probes are one gap apart and never held; playInputs keeps its signature for its six callers; every start now reaches ready within a probe or two of the pane showing it.
