# Card 4 — startup-probe-cadence

**Edit:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TrustPromptThenReady`
- `internal/shuttleengine#TestStartup_TickCap`

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsCounted` -> `func playInputsCounted(reed ReedOps, guid string, inputs []PaneInput) (int, error)`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#Run.Wait`

**Intent:**
Every shuttle start sees a ready provider within about a second of it turning ready, instead of after the 10-second probe interval (decision D5 as amended by the parent's addition), while a startup-gate dismissal is still played at most once per dismissal gap (decision D6).
The two land together because the faster probe is safe only behind the dismissal throttle.

`dismissalGap(cfg)` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Wait` floors it: 10 s under the shipped template.

Probe cadence, in `Run.awaitStartup`, following `PATTERN-daemon-wakeups`' backoff rule as written; the background is not amended:
- The base sleep is `pollInterval(cfg)` and the ceiling is `dismissalGap(cfg)`.
- The first probe and every probe whose pane capture differs from the previous probe's are followed by a base sleep.
  Any other probe, a failed capture or a failed status included, is followed by double the previous sleep, capped at the ceiling.
  The capture compared is `run.lastStartupCapture` before and after the probe.
- The tick cap is `startupTickCap(window, pollInterval(cfg))`.
- Each probe still runs `checkLivenessTick`, and every negative answer still comes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist` (`PATTERN-completion-signal`); no return statement is added or removed, so the audited-return scan in `completionsignal_enforcement_test.go` is unchanged.
- No sub-second literal is added; the sleeps derive from the floored config interval.
- The "Probe cadence" paragraph of `awaitStartup`'s doc comment is rewritten for the backoff and the dismissal throttle, one sentence per line.
- What the change lets through: per-second `reed status` calls and pane captures while the pane keeps changing during startup, bounded by `startup_timeout_s`, the tick cap and the first ready probe.
  A static boot screen backs off to the ceiling; Claude's boot redraws reset it, so a ready provider still shows within the first one or two probes.
`Wait`'s own probe cadence is unchanged.

Dismissal throttle, in `Run.checkLivenessTick`'s `StartupTrustPrompt` branch:
- `Run` gains a field `lastDismissal time.Time`, the run clock's time of the last dismissal play that sent at least one input, zero until one does.
- The branch plays the engine's trust-dismiss sequence only when `lastDismissal` is zero or the run clock's now is at or past `lastDismissal` plus `dismissalGap(cfg)`.
  A probe inside the gap that still sees a gate plays nothing, logs at Debug that the dismissal is held, and classifies the startup window as before.
- It plays through `playInputsCounted`, which plays inputs as `playInputs` does and returns how many it delivered before the first failure.
  `playInputs` becomes `playInputsCounted` with the count dropped, so its callers keep their behavior.
- A play that delivered at least one input, failed partway or not, sets `lastDismissal` to the probe's now.
  An empty sequence, or a play whose first input failed, leaves it, so the next probe is free to dismiss.
- The existing Warn on a failed play and Info on a successful one stay.
- Bound: the throttle only removes key presses and never adds one; the startup deadline and the tick cap still end a gate that never clears.
  `Wait`'s startup probe runs at exactly the gap, so under the at-or-past rule the throttle never holds a dismissal there.

`template.yaml`'s comment on `liveness_every_n_polls` adds that the same multiple of `poll_interval_ms` spaces startup-gate dismissals; no config key is added.

Tests, all on the injected fake clock, none waiting out an interval:
- `TestStartup_ReadyProbeTiming` gains a per-row capture queue.
  The row "probe cadence matches Wait" becomes "a changing capture is probed every poll interval": `PollIntervalMS: 1000, LivenessEveryNPolls: 10`, script Pending, Pending, Ready over three distinct captures, wanting 3 status calls and 2 s elapsed.
  A row "an unchanged capture doubles the sleep up to the dismissal gap": one constant capture, script five Pending then Ready, wanting 6 status calls and 1+2+4+8+10 = 25 s.
  A row "a changed capture resets the sleep to the poll interval": captures a, a, a, b, script four Pending then Ready, wanting 5 status calls and 1+2+4+1 = 8 s.
  The doc comment and the `//testtiming:keep` reason are reworded to the backoff.
- `TestStartup_TrustPromptThenReady` becomes a table whose first row is today's scenario, unchanged in its assertions.
  A row "a gate on consecutive one-second probes is dismissed once per dismissal gap": `LivenessEveryNPolls: 10`, twelve distinct gate captures so the probes stay one second apart, script twelve `StartupTrustPrompt` then Ready, wanting exactly two `Enter` keys (at 0 s and 10 s).
  A row "a gate that never clears still ends the start at the startup deadline as died": `StartupTimeoutS: 5`, `LivenessEveryNPolls: 10`, distinct gate captures and a script of `StartupTrustPrompt` only, wanting an `ErrNotStarted` error, a died outcome in run.json and exactly one `Enter` key.
  A row "a play whose first key fails leaves the next probe free to dismiss": `SendKeyErr` set, distinct gate captures, script three `StartupTrustPrompt` then Ready, wanting an `Enter` attempt on each of the three gate probes.
- `TestStartup_TickCap` computes its wanted tick count as `startupTickCap(startupTimeout, pollInterval(cfg))`.

**ImpactSummary:** Changes the startup probe's sleep schedule and gates the trust-dismiss play behind a per-run gap, both inside `shuttleengine`'s startup step; `playInputs`' callers keep their behavior and `Wait` is unchanged.

**Commit:** 4: Startup probes back off from one poll interval and space gate dismissals
