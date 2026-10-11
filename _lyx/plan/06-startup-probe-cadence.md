# Card 6 — The startup probe runs at the poll interval with backoff, and gate dismissals are spaced by the dismissal gap

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (bool, error)`
- `plan:internal/shuttleengine#twoKeyGateEngine` -> `type twoKeyGateEngine struct`
- `plan:internal/shuttleengine#twoKeyGateEngine.TrustDismissSequence` -> `func (e *twoKeyGateEngine) TrustDismissSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#enterFailsReed` -> `type enterFailsReed struct`
- `plan:internal/shuttleengine#enterFailsReed.SendKey` -> `func (r *enterFailsReed) SendKey(guid, key string) error`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestStartup_UndismissableGateUntilWindowExpires`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#Run.Wait`

**Intent:**
Decisions D5 and D6, with D5's cadence as the parent amended it after the Discussion: every shuttle start passes its startup probe within about a second of the provider becoming ready, instead of the 10-second sleep that makes every start take about 11 seconds, while a startup gate still gets no second key before Claude redraws.

`dismissalGap`, in `wait.go` beside `pollInterval`, returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, with the multiple floored to 1 exactly as `Wait` floors it: 10 s under the shipped template.

D5 and D6 share one card because neither stands alone: under today's cadence the dismissal gap equals the probe interval, so the throttle never holds and no test could reach it, while the faster cadence without the throttle would replay a gate key every second.

Probe cadence, in `Run.awaitStartup`: the pre-ready probe adopts `PATTERN-daemon-wakeups`' backoff rule.
`awaitStartup` has a deadline and a tick cap of its own, so it is a bounded wait; its 90 s window is over the background's one-minute exemption, and the parent's decision applies the backoff rule to it rather than leaving it unplaced.
The sleep after a probe starts at `pollInterval(cfg)`, doubles while the probe's capture equals the previous probe's, is capped at `dismissalGap(cfg)`, and returns to `pollInterval(cfg)` whenever the capture changes.
The first probe counts as a change.
"The capture" is `run.lastStartupCapture` after the probe, so a failed capture, which leaves it unchanged, counts as no change.
Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture; every negative answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist` (`PATTERN-completion-signal`).
The tick cap is `startupTickCap(window, pollInterval(cfg))`, since a probe is never closer than the base interval; the deadline and the cap still end a pane that never turns ready.
No sub-second literal is added, and `Wait`'s own probe cadence is unchanged.
`awaitStartup`'s probe-cadence comment is rewritten to state the backoff and why the dismissal gap, not the probe interval, now carries the stray-keypress protection; `startupTickCap`'s comment says it takes the base poll interval.
`doc.go`'s startup paragraph states the cadence: the poll interval, backing off to the dismissal gap while the pane does not change.
A ready provider still shows within one or two probes, because Claude's boot redraws the pane and resets the backoff.

Dismissal gap, in `Run.checkLivenessTick`: `Run` gains a field holding the run clock's time of the last startup-gate play that sent a key, zero when none has.
On `StartupTrustPrompt`, a probe at or past one `dismissalGap(cfg)` after that time, or with none recorded, plays the engine's trust-dismiss sequence; a probe inside the gap plays nothing and classifies the startup window as before.
The first gate a run sees is dismissed on the probe that sees it.
`playInputsSent`, in `run.go`, is `playInputs` reporting whether its first input was delivered; `playInputs` keeps its signature and delegates to it, so its other callers are untouched.
A trust-dismiss sequence is keys only, so for the throttle "first input" is D6's "first key"; reporting it for any input keeps one helper for both input kinds.
Any play whose first key was sent starts the gap, whether a later key then failed or not; an empty sequence or a play whose first key failed leaves the next probe free to dismiss.
The existing Warn on a failed play and Info on a dismissal stay.
Bound: the throttle only removes key presses and never adds one; the startup deadline and tick cap still end a gate that never clears.
`Wait`'s startup probe already runs one gap apart, so under the at-or-past rule the throttle never holds a dismissal there.

`template.yaml`'s comment on `liveness_every_n_polls` adds that it also spaces startup-gate dismissals; no config key is added.

`PATTERN-daemon-wakeups` is not amended: the parent's post-Discussion decision places `awaitStartup` under its backoff rule, superseding D5's exemption and the Constraints line that cites it.

Tests, all on the fake or frozen clock:
- `TestStartup_ReadyProbeTiming` gains a per-row capture queue.
  The "probe cadence matches Wait" row becomes "a changing pane is probed every poll interval": `PollIntervalMS 1000`, `LivenessEveryNPolls 10`, script pending, pending, ready over three distinct captures, three status calls and exactly 2 s elapsed, which pins ready on the first probe after the pane shows ready.
  A row "an unchanging pane backs off to the dismissal gap" scripts six pending probes then ready over one constant capture: seven status calls and exactly 1+2+4+8+10+10 = 35 s.
  A row "a changed capture resets the backoff" scripts four pending probes then ready over captures `boot`, `boot`, `boot`, `drawn`: five status calls and exactly 1+2+4+1 = 8 s.
- `TestStartup_TickCap`'s expected count becomes `startupTickCap(timeout, pollInterval(cfg))`.
- `TestStartup_UndismissableGateUntilWindowExpires` becomes a table of gates that never clear, each ending at the startup deadline as an `ErrNotStarted` died start with the strand removed once, over `PollIntervalMS 1000`, `LivenessEveryNPolls 10`, `StartupTimeoutS 25`, `RunTimeoutMin 5` and a distinct gate capture per probe so the backoff stays at one second.
  Its name stays: every row is a gate whose dismissal never clears it before the window expires, which is what "undismissable" names, and keeping it keeps the test's `testtiming:keep` record.
  Its rows: the existing undismissable gate, which sends no key and logs no dismissal; a dismissable gate, dismissed exactly three times, once per gap (at 0, 10 and 20 s); a gate whose first key fails (`fakeReed.SendKeyErr`), whose dismissal is attempted on every probe, as many `SendKey` calls as `Status` calls, which a throttle wrongly started by the failed key would cut to three; and a gate whose two-key sequence fails on its second key, whose first key is sent exactly three times.
  `twoKeyGateEngine` and `enterFailsReed` live in `startup_test.go` beside `undismissableEngine` and `flakyStatusReed`: `twoKeyGateEngine` embeds `*fakeEngine` and answers `Down` then `Enter`; `enterFailsReed` embeds `*fakeReed` and fails every `Enter` after recording it.
  Its `testtiming:keep` reason names the throttle rows too.

**ImpactSummary:** Package-private only: `awaitStartup`'s sleep and tick cap, one dismissal guard in `checkLivenessTick` shared with `Wait`'s startup probe (which never trips it), and a `playInputs` split that leaves its other callers untouched; every Start, StartGated and RunGated caller starts about 10 s sooner with no API change.
