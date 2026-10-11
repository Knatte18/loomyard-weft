# Card 4 — The startup probe backs off from the poll interval, and gate dismissals keep their gap

**Create:**
- `plan:internal/shuttleengine#startupDismissalGap` -> `func startupDismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (int, error)`
- `plan:internal/shuttleengine#TestStartup_DismissalGap` -> `func TestStartup_DismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#keyFailReed` -> `type keyFailReed struct`
- `plan:internal/shuttleengine#keyFailReed.SendKey` -> `func (r *keyFailReed) SendKey(guid, key string) error`
- `plan:internal/shuttleengine#sequenceDismissEngine` -> `type sequenceDismissEngine struct`
- `plan:internal/shuttleengine#sequenceDismissEngine.TrustDismissSequence` -> `func (e *sequenceDismissEngine) TrustDismissSequence(capture string) []PaneInput`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/template.yaml`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`

**Intent:** Decision D5 as superseded by the parent's backoff decision, and decision D6: a start sees a ready provider within about a second, while a startup gate still gets at most one dismissal per dismissal gap.
The two land together because the faster probe is what makes the dismissal throttle necessary.

`startupDismissalGap` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Wait` floors it: 10 s under the shipped template.

`Run.awaitStartup`'s pre-ready probe follows `PATTERN-daemon-wakeups`'s backoff rule, with base `pollInterval(cfg)` and ceiling `startupDismissalGap(cfg)`.
The sleep after the first probe is the base.
Before each probe it keeps the `run.lastStartupCapture` it had; a probe that recorded a different capture makes the next sleep the base, and any other probe, an unchanged capture, a failed capture or a status error, makes it twice the previous sleep, capped at the ceiling.
Claude's boot redraws the pane and so resets the backoff, so a ready provider still shows within the first one or two probes.
The tick cap becomes `startupTickCap(window, pollInterval(cfg))`, which still bounds the loop under a clock that never advances, and the deadline, window and every exit are unchanged.
`PATTERN-daemon-wakeups` is not amended, and no sub-second literal is added.
The probe-cadence paragraph of `awaitStartup`'s doc comment is rewritten one sentence per line: the backoff, its base and ceiling, the reset on a changed capture, and that dismissals are spaced by the gap rather than the probes; the paragraph stating it probes at Wait's cadence and "never faster" goes.
`startupTickCap`'s doc comment names the base poll interval instead of the probe interval.

`Run` gains a `lastDismissalAt time.Time` field, zero until a dismissal starts the gap.
`playInputsSent` plays inputs as `playInputs` does today and returns how many inputs it delivered before the first failure; `playInputs` becomes `_, err := playInputsSent(...)`, so its other callers are unchanged.
`Run.checkLivenessTick`'s `StartupTrustPrompt` arm plays the engine's trust-dismiss sequence only when `lastDismissalAt` is zero or the run clock's now is at or past `lastDismissalAt` plus `startupDismissalGap(cfg)`.
It reads now before the play, and a play that delivered at least one input, failed partway or not, sets `lastDismissalAt` to that now; an empty sequence, or a play whose first input failed, leaves it, so the next probe may dismiss.
A probe inside the gap plays nothing and calls nothing on the engine's dismissal, and every arm still ends in `classifyStartupWindow`, so a gate that never clears still ends at the startup deadline as died (`PATTERN-completion-signal` unchanged).
Bound: the throttle only removes key presses and never adds one.
`Wait`'s startup probe runs at the gap already, so the at-or-past rule never holds a dismissal there.
`checkLivenessTick`'s doc comment states the throttle: the gap, the at-or-past rule and what starts it; the comment on the dismissal's Info line, "at most once or twice per run", says a dismissal is played at most once per gap.

`internal/shuttleengine/template.yaml`'s `liveness_every_n_polls` comment says it also spaces startup-gate dismissals, at that many polls; no config key is added.

Tests, all on the fake clock:
- `TestStartup_ReadyProbeTiming` gains a `captures` field fed to the fake reed's `CaptureQueue`, and its "probe cadence matches Wait" row is replaced, under `PollIntervalMS 1000` and `LivenessEveryNPolls 10`, by: "a changing pane is probed every poll interval" (captures `a`, `b`, `c`; pending, pending, ready; 3 status calls; 2 s); "an unchanged pane backs off to the dismissal gap" (no captures, so every capture is empty; five pending then ready; 6 status calls; 1+2+4+8+10 = 25 s); "a changed capture resets the backoff" (captures `a`, `a`, `a`, `b`; four pending then ready; 5 status calls; 1+2+4+1 = 8 s).
  Its doc comment and its `//testtiming:keep` reason name the backoff instead of "two probe intervals".
  The `startup_test.go` file comment's "probe-cadence-matches-Wait assertion" becomes the backoff assertion.
- `TestStartup_TickCap` computes `wantTicks` from `pollInterval(cfg)`.
- `TestStartup_DismissalGap` is table-driven over `PollIntervalMS 1000`, `LivenessEveryNPolls 10`, `StartupTimeoutS 30`, `RunTimeoutMin 5`, with the multiple a row field: the startup script stays `StartupTrustPrompt` and the capture queue holds a distinct gate screen per probe, so every sleep stays at the base and probes land every second from 0 s to 31 s, when the window expires.
  Rows: "a gate that never clears is dismissed once per gap" wants 4 `Enter` keys (0, 10, 20 and 30 s); "a play whose first key fails leaves the next probe free" sets `SendKeyErr` and wants an `Enter` attempt on every one of the 32 probes; "a play that fails after its first key starts the gap" uses `sequenceDismissEngine` answering `Enter` then `Tab` and `keyFailReed` failing `Tab` only, and wants 4 `Enter` keys; "a zero multiple is floored to one poll" sets `LivenessEveryNPolls 0`, so the gap is one second, and wants an `Enter` on every one of the 32 probes.
  Every row wants an error wrapping `ErrNotStarted`, run.json's Outcome died, and 31 s of virtual time.
  `keyFailReed` embeds `*fakeReed`, records the call through it and returns an error for its `failKey`; `sequenceDismissEngine` embeds `*fakeEngine` and answers its `inputs`.
  The new top-level test fits the package's row in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** `awaitStartup`'s sleeps and tick cap change only under `liveness_every_n_polls` above 1, and `fastConfig`/`shortStartupConfig` tests keep their timing since their ceiling equals the base, and a Start-driven test under `gateConfig` or `sparseProbeConfig` whose startup is still pending now probes every second up to the gap instead of once per gap, which no test's timing pins but the card's own package run confirms; `checkLivenessTick`'s throttle never holds in `Wait`; `playInputs`'s callers in `run.go` and `send.go` are unchanged.

**Commit:** 4: Startup probe backs off from the poll interval; gate dismissals keep their gap
