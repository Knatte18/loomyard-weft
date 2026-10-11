# Card 4 — startup-probe-cadence

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (sent bool, err error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#gateSequenceEngine` -> `type gateSequenceEngine struct`
- `plan:internal/shuttleengine#nthKeyFailReed` -> `type nthKeyFailReed struct`

**Edit:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/template.yaml`

**Intent:** The startup probe sees a ready provider within about a second, and a startup gate is dismissed at most once per dismissal gap (D5 as the parent amended it to the backoff rule, and D6).
`dismissalGap(cfg)` is `pollInterval(cfg)` times `cfg.LivenessEveryNPolls` floored to 1 exactly as `Wait` floors it, 10 seconds under the shipped template.
`awaitStartup` probes at `pollInterval(cfg)` instead of the gap and computes its tick cap from `pollInterval(cfg)`; each probe still runs `checkLivenessTick`, one reed status and, before ready, one pane capture.
Its sleep follows `PATTERN-daemon-wakeups`: the base is `pollInterval(cfg)`, it doubles while the pane capture is unchanged from the previous probe up to the ceiling `dismissalGap(cfg)`, and it returns to the base whenever the capture changes; the first probe counts as changed, and a probe whose capture failed leaves `run.lastStartupCapture` as it was, so it counts as unchanged.
`PATTERN-daemon-wakeups` itself is not amended and no sub-second literal is added; `startupTickCap` keeps its signature, its comment now says the interval is the poll interval.
`awaitStartup`'s probe-cadence paragraph is rewritten, one sentence per line, to state the poll-interval base, the backoff and why the gap now spaces dismissals instead of probes.
The deadline and tick-cap exits, and every `allOutputFilesExist` call and negative return in `awaitStartup`, are unchanged (`PATTERN-completion-signal`), so `completionsignal_enforcement_test.go` needs no edit.
`playInputsSent` plays the inputs as `playInputs` does and also reports whether at least one input was delivered, true after the first successful `SendKey` or `SendText` even when a later one fails; `playInputs` delegates to it and drops the flag, so its callers and signature are untouched.
`Run` gains a `lastDismissAt time.Time` field, zero until a startup-gate play sends an input.
In `checkLivenessTick`'s `StartupTrustPrompt` case the engine's `TrustDismissSequence` is read and played only when `lastDismissAt` is zero or `run.clock.Now()` is at or past `lastDismissAt` plus `dismissalGap(cfg)`.
A play for which `playInputsSent` reports sent, failed partway or not, sets `lastDismissAt` to now; an empty sequence or a play whose first input failed leaves it unset, so the next probe may dismiss.
A probe inside the gap that still sees a gate plays nothing and falls through to `classifyStartupWindow` as before, and the first gate a run sees is dismissed on the probe that sees it.
What the change can skip or let through: the throttle only removes key presses and never adds one, so a gate that never clears still ends the start at `startup_timeout_s` as died, and the backoff is bounded by the same deadline and tick cap.
`Wait`'s own startup probe runs at least one gap apart with the same floored multiple, so under the at-or-past rule the throttle never holds a dismissal there; an attached `Run` starts with a zero `lastDismissAt`.
`template.yaml`'s `liveness_every_n_polls` comment adds that the value also spaces startup-gate dismissals, with no new config key, and `doc.go`'s startup paragraph states the poll-interval probe, the backoff ceiling and the dismissal gap.
`TestStartup_ReadyProbeTiming` replaces its "probe cadence matches Wait" row, which pins the old twenty-second elapse, with rows on the same virtual clock and `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60, RunTimeoutMin: 5}`, driven by a new `captures` row field feeding the fake reed's capture queue: a changing pane (pending, pending, ready over three distinct captures) elapses two seconds with three status calls; an unchanged pane (six pending then ready, empty captures) elapses 35 seconds as sleeps of 1, 2, 4, 8, 10 and 10 seconds with seven status calls; a pane that changes mid-wait (pending four times then ready over captures a, a, b, b, c) elapses 6 seconds, showing the reset; the existing ready-on-first-probe row stays.
Its doc comment and its `//testtiming:keep` reason, which says "exactly two probe intervals otherwise", are rewritten to match, as is the `startup_test.go` header sentence that lists a "probe-cadence-matches-Wait assertion".
`TestStartup_TickCap` computes `wantTicks` from `pollInterval(cfg)` alone.
`TestStartup_GateDismissalGap` is one table-driven test on `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, RunTimeoutMin: 5}` with a per-row `StartupTimeoutS`, a fake clock, a trust-prompt `StartupScript`, a status queue holding the strand live, and, except for the constant-capture row, a capture queue of distinct strings (`gate 0`, `gate 1`, ... forty of them) so the backoff stays at one second and probes fall at t = 0, 1, 2, ... seconds.
`gateSequenceEngine` embeds `*fakeEngine` and overrides `TrustDismissSequence` to return a row-chosen input list while counting its calls; `nthKeyFailReed` embeds `*fakeReed` and overrides `SendKey` to fail on the one attempt number a row names, and otherwise delegates so the key is recorded.
The rows: with an Enter sequence and a 25-second window, the Enter is sent at t = 0, 10 and 20 (three sequence calls, three keys), the dismissal at exactly t = 10 pinning the at-or-past rule, and the start ends `ErrNotStarted` with run state outcome `died` at the deadline; with an Enter sequence on a reed whose `SendKeyErr` is set (every key fails, and `fakeReed` records each attempt) and a 5-second window, the sequence is read at every probe from t = 0 to 6, seven calls and seven recorded attempts; with a two-key sequence on `nthKeyFailReed` failing its second attempt and a 5-second window, one call and one recorded key, since the failed play still started the gap; with an empty sequence and a 5-second window, seven calls and no key; and with an Enter sequence, one constant gate capture so the backoff climbs, and a 30-second window, the Enter is sent at t = 0, 15, 25 and 35 (four calls, four keys) before the start ends died, showing the throttle holds under the backoff's growing sleeps.

**ImpactSummary:** `awaitStartup` is called only from `start`, and `checkLivenessTick` from `awaitStartup` and `Wait`; the throttle can only suppress a repeated dismissal, and the shipped template's cadence leaves `Wait` unchanged.

**Commit:** `4: startup-probe-cadence`
