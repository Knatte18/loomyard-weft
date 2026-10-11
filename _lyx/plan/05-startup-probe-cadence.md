# Card 5 — The startup probe backs off from one poll interval, and a gate dismissal is played at most once per dismissal gap

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (sent int, err error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#Run.Wait`

**Intent:**
Decision D5 as superseded by the parent's backoff addition, and decision D6 with the parent's addition on partial plays.
Today `awaitStartup` sleeps `pollInterval(cfg) × liveness_every_n_polls`, 10 s under the shipped template, between probes, so every shuttle start takes about 11 seconds although the provider is ready within about a second.
D5's exemption from `PATTERN-daemon-wakeups` is dropped: the probe follows the background's backoff rule as written, and `pattern/PATTERN-daemon-wakeups.md` is not amended.
The throttle lands in the same commit as the faster probe, so no commit probes every second while replaying a dismissal on every probe.

`dismissalGap` returns `pollInterval(cfg) × cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Run.Wait` floors it: 10 s under the shipped template.
It is the backoff's named ceiling and the dismissal throttle's gap.

`Run.awaitStartup`'s probe cadence:
- The base is `pollInterval(cfg)`, and the tick cap is `startupTickCap(window, pollInterval(cfg))`, which bounds the backed-off loop from above.
- After each probe, the next sleep is the base when the probe's pane capture differs from the previous probe's (`run.lastStartupCapture` read before and after `checkLivenessTick`), and otherwise doubles the previous sleep up to `dismissalGap(cfg)`.
  The first probe compares against no capture, so a first successful capture counts as a change; a failed capture leaves `lastStartupCapture` as it was and counts as no change.
- Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture; no sub-second literal is added.
Claude's boot redraws the pane and so resets the backoff, which keeps a ready provider visible within one or two probes.
Rewrite `awaitStartup`'s "Probe cadence" doc paragraph, one sentence per line, to state the base, the doubling while the capture is unchanged, the reset on a change, the ceiling, and that the dismissal throttle, not the cadence, now guards against a second key into the next gate.
The Completion Signal Invariant is untouched: every negative answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`.

The dismissal throttle:
- `playInputsSent` holds today's `playInputs` loop and also returns how many inputs were sent before it returned; `playInputs` returns `playInputsSent`'s error alone, so its callers are unchanged.
- `Run` gains a field holding the run clock's time of the last startup-gate play that sent at least one key, zero when none has.
- In `Run.checkLivenessTick`'s `StartupTrustPrompt` branch, the dismiss sequence is played only when that field is zero or `run.clock.Now()` is at or past one `dismissalGap(cfg)` after it.
  The field is set to the probe's time whenever `playInputsSent` reports at least one input sent, whether the play then failed or not, since a failed play can leave keys in the pane.
  An empty sequence, or a play whose first input failed, leaves the field as it was, so the next probe may dismiss.
- A probe inside the gap that still sees a gate plays nothing, logs nothing, and classifies the startup window as before.
- The first gate a run sees is dismissed on the probe that sees it.
Bound: the throttle only removes key presses and never adds one; the startup deadline and the tick cap still end a gate that never clears.
`Run.Wait`'s own startup probe for a re-attached unstarted run already runs at the dismissal gap, at least one gap apart, so under the at-or-past rule the throttle never holds a dismissal there.

`template.yaml`: the `liveness_every_n_polls` comment adds that the same multiple of `poll_interval_ms` spaces startup-gate dismissals and caps the startup probe's backoff; no key is added.
`doc.go`: the startup-probe paragraph adds one sentence each for the startup probe's cadence and the dismissal gap.

Tests, on the fake clock, never waiting out an interval:
- `TestStartup_ReadyProbeTiming`: the rows gain a capture queue.
  Rename the `probe cadence matches Wait` row and change it to distinct pending captures then ready, under `PollIntervalMS: 1000, LivenessEveryNPolls: 10`: ready on the third probe after exactly two base intervals, 2 s.
  Add a row whose pending capture repeats unchanged before the ready one: the sleeps double from 1 s to the 10 s ceiling and hold there, and the elapsed time is their sum.
  Add a row whose pending capture changes mid-way: the sleep returns to 1 s on the change.
- `TestStartup_TickCap`: the expected tick count is `startupTickCap` over `pollInterval(cfg)`.
- `TestStartup_GateDismissalGap`, a table over a pane that shows a gate on every probe, with the capture changing on every probe so the probe stays at the 1 s base, under `PollIntervalMS: 1000, LivenessEveryNPolls: 10` and a startup window of 25 s:
  - a gate that never clears gets one dismissal per gap, Enter at 0 s, 10 s and 20 s on the run clock, and the start still ends at the startup deadline wrapping `ErrNotStarted` with outcome died;
  - a play whose first key fails starts no gap, so the next probe, 1 s later, plays again;
  - a two-key play whose second key fails starts the gap, so no key is sent until 10 s later.
  The run clock's time at each `SendKey` call is recorded by the test's reed double.
  The package has room for this one new top-level test in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** every shuttle start probes from a 1 s base backing off to the 10 s gap while the pane is unchanged, and replays a startup-gate dismissal at most once per gap; `playInputs`' callers, `Wait`'s cadence and the template's keys are unchanged.

**Commit:** 5: Probe startup from one poll interval and space gate dismissals by the gap
