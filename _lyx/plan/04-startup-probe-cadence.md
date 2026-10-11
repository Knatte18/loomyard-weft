# Card 4 — startup-probe-cadence

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsCounted` -> `func playInputsCounted(reed ReedOps, guid string, inputs []PaneInput) (sent int, err error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`

**Edit:**
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`

**Intent:**
Every shuttle start sees its provider ready within about a second of the pane showing it, instead of on the next 10-second probe, while a startup-gate dismissal is still replayed at most once per 10 seconds.
This is decision D5 as the parent's later addition supersedes it (`awaitStartup` backs off under `PATTERN-daemon-wakeups`, which is not amended), together with D6 and its later addition on when a play starts the gap.
The two land in one card because a one-second probe without the throttle would replay a gate's keys every second, the stray-keypress hazard the 10-second cadence existed to prevent.

`dismissalGap` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Run.Wait` floors it: 10 s under the shipped template.

`playInputsCounted` holds `playInputs`'s loop and also returns how many inputs were delivered before it returned, so a failure on the first input reports 0.
`playInputs` keeps its signature and returns `playInputsCounted`'s error, so none of its callers changes.

`Run` gains `lastGateDismissal time.Time`, the run clock's time of the last startup-gate play that delivered at least one input, zero before any.
In `Run.checkLivenessTick`'s `StartupTrustPrompt` branch, the trust-dismiss sequence is played only when `lastGateDismissal` is zero or the run clock's now is at or past `lastGateDismissal` plus `dismissalGap(cfg)`.
A play that delivered at least one input, whether it then failed or not, sets `lastGateDismissal` to the time read before the play; an empty sequence or a play whose first input failed leaves it unchanged, so the next probe may dismiss again.
A probe inside the gap that still sees a gate plays nothing, logs one Debug line naming the strand, and classifies the startup window as before.
The first gate a run sees is dismissed on the probe that sees it.
`Run.Wait`'s own startup probe runs at the dismissal gap with at least one gap between probes, so under the at-or-past rule the throttle never holds a dismissal there.
Bound: the throttle only removes key presses and never adds one; a gate that never clears still ends the start at `startup_timeout_s` as died, and the tick cap still bounds the loop.

`Run.awaitStartup` probes before ready at a backoff under `PATTERN-daemon-wakeups`: the sleep starts at the base `pollInterval(cfg)`, a probe whose recorded capture (`run.lastStartupCapture`) differs from the previous probe's resets it to the base, and a probe whose capture is unchanged, failed or absent doubles it up to the ceiling `dismissalGap(cfg)`.
The first probe has no previous capture and sleeps the base.
Claude's boot redraws the pane on every step, which keeps the sleep at the base, so a ready provider is seen on the first one or two probes after it draws its box.
Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture; no sub-second literal is added.
The tick cap is `startupTickCap(window, pollInterval(cfg))`, computed from the base, so a run of unchanged captures that backs off runs fewer probes than the cap allows and the deadline ends it.
What the cadence change lets through is more reed status calls and pane captures per start, at most one per second, bounded by the first ready probe, `startup_timeout_s` and the tick cap.
`awaitStartup`'s probe-cadence doc comment is rewritten, one sentence per line, to state the backoff, its base and ceiling, the dismissal throttle and why the two together replace the old fixed probe interval;
`startupTickCap`'s doc comment says its interval is the base poll interval.

`template.yaml`'s comment on `liveness_every_n_polls` adds that the same multiple of `poll_interval_ms` spaces startup-gate dismissals and caps the startup probe's backoff; no key is added.

`TestStartup_ReadyProbeTiming`'s "probe cadence matches Wait" row is replaced by rows under `PollIntervalMS: 1000, LivenessEveryNPolls: 10`, with a capture queue column on the table:
captures that change on every probe and a pending, pending, ready script reach ready on the third probe after exactly 2 s of virtual sleep;
an unchanged capture over the same script sleeps 1 s then 2 s;
an unchanged capture over six pending probes then ready sleeps 1, 2, 4, 8, 10 and 10 s, so the backoff stops at the 10 s ceiling.
Its doc comment and `//testtiming:keep` line are reworded to the backoff.
`TestStartup_TickCap` computes its wanted tick count from `pollInterval(cfg)` alone.

`TestStartup_GateDismissalGap` is table-driven under `PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 30` on a fake clock, with an engine that classifies every probe `StartupTrustPrompt` over a capture that differs on every probe, so the probe stays at one second, and a reed double that records the virtual time of each `SendKey` call.
Rows: a one-key Enter sequence plays at 0, 10, 20 and 30 s and the start then ends `ErrNotStarted` with a died record when the window closes;
a sequence whose first key always fails is attempted on every probe;
a two-key sequence whose second key always fails plays its first key at 0, 10, 20 and 30 s, so a partial play starts the gap.
Each row asserts the exact recorded key times, derived from that schedule.

**ImpactSummary:** start-time readiness latency drops from one 10 s probe to about one second; gate keys keep their 10 s spacing; `playInputs` callers are untouched; `Run.Wait` keeps its own cadence and is never throttled; tests with `LivenessEveryNPolls` above 1 that pass through a not-yet-ready startup see the backoff's sleeps instead of one fixed interval.

**Commit:** `4: Startup probes back off from one second; gate dismissals keep their gap`
