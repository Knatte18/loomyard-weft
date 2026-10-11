# Card 4 — `awaitStartup` probes at the poll interval and spaces gate dismissals

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (int, error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#gateKeysEngine` -> `type gateKeysEngine struct`
- `plan:internal/shuttleengine#gateKeysEngine.TrustDismissSequence` -> `func (e *gateKeysEngine) TrustDismissSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#failingEnterReed` -> `type failingEnterReed struct`
- `plan:internal/shuttleengine#failingEnterReed.SendKey` -> `func (r *failingEnterReed) SendKey(guid, key string) error`

**Edit:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `internal/loomcli#integrationDriverShuttleConfig`
- `internal/websterengine#AcquireStateMutation`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#Run.classifyStartupWindow`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#newFakeClock`

**Intent:**
D5 with its added backoff, and D6 with its added "any play that sent a key starts the gap", of the decision record.
They land together because probing every second is only safe once a dismissal is spaced by the gap, and the spacing is only observable at the faster cadence.

Dismissal gap (D6).
`dismissalGap(cfg)` is `pollInterval(cfg)` times `LivenessEveryNPolls` floored to 1, the multiple exactly as `Wait` floors it: 10 s under the shipped template.
`Run` gains `lastGateDismissAt time.Time`, zero until a play sent at least one input.
In `checkLivenessTick`'s `StartupTrustPrompt` case, the engine's `TrustDismissSequence(capture)` is played only when `lastGateDismissAt` is zero or `run.clock.Now()` is at or past `lastGateDismissAt + dismissalGap(cfg)`; the first gate a run sees is dismissed on the probe that sees it.
The play goes through `playInputsSent`, which is `playInputs`' loop returning how many inputs were sent; `playInputs` becomes a call to it that drops the count, keeping its signature and its other callers.
A play that sent one or more inputs, failed partway or not, sets `lastGateDismissAt` to the clock's now; an empty sequence, or a play whose first input failed, leaves it unset so the next probe may dismiss.
A probe inside the gap that still sees a gate plays nothing and falls through to `classifyStartupWindow` as before, so the startup deadline and the tick cap still end a gate that never clears as died.
What the throttle can now skip: a dismissal on a probe inside the gap; it only removes key presses, never adds one.
The input that reaches that branch after every upstream check passes is a gate capture on a probe less than one gap after a played dismissal, which needs probes closer than the gap, and the tests below reach it.
`Wait`'s own startup probe runs one gap apart with the same floored multiple, so under the at-or-past rule the throttle never holds a dismissal there; row one of `TestStartup_GateDismissalGap` pins the at-or-past comparison itself, the one Wait's boundary relies on, by requiring the dismissal at exactly 10 s and 20 s.

Probe cadence (D5 plus backoff).
`awaitStartup` sleeps, after a probe that neither returned nor expired, `pollInterval(cfg)` when it is the first probe or the pane capture changed since the previous probe (`run.lastStartupCapture` before and after `checkLivenessTick`), and otherwise double the previous sleep capped at `dismissalGap(cfg)`.
That is the `PATTERN-daemon-wakeups` backoff: base the poll interval, doubling while nothing changes, ceiling the gap, reset to the base on a change, and the one-second floor holds since `pollInterval` floors.
A ready provider shows within the first one or two probes because Claude's boot redraws the pane and resets the backoff.
`maxTicks` is `startupTickCap(window, pollInterval(cfg))`, the base interval, so the count cap still bounds every probe a clock that never advances could make; `startupTickCap`'s comment names the base interval instead of the dismissal-gap multiple.
`PATTERN-daemon-wakeups` is not amended, and no sub-second literal is added.
What the backoff can now skip: a probe for up to the gap after a static boot screen turns ready; it is bounded by the gap, `startup_timeout_s` and the tick cap, and a changed capture resets it.
No exit of `awaitStartup` or `checkLivenessTick` is added or reworded, so every negative startup answer still comes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`, and the counts `completionsignal_enforcement_test.go` audits stay as they are.
The rewritten probe-cadence comment on `awaitStartup`, `checkLivenessTick`'s doc and `wait.go`'s file doc keep one sentence per line.

Docs in the same commit: `template.yaml`'s comment on `liveness_every_n_polls` says it also sets the minimum gap between startup-gate dismissals and the startup probe's backoff ceiling, with no key added; `doc.go` states the startup cadence and gap beside its paragraph on the startup probe.
`AcquireStateMutation`'s doc comment drops its "typically one or two probe intervals, about 5–10 s under the shipped config" figure, since a ready provider is now seen within about a second; the bound it names, `startup_timeout_s`, stays.
`startup_test.go`'s file header comment, which names "the probe-cadence-matches-Wait assertion", is reworded in the same edit as the `TestStartup_ReadyProbeTiming` change to name the cadence and dismissal-gap tests; the file has no symbol for its header, and a file path beside its members is refused.
`integrationDriverShuttleConfig` keeps its `liveness_every_n_polls: 1` override, and its comment now says why: the stub's boot screen is static until its ready marker, so a backoff ceiling above the base interval would make the probe see the marker late.

Tests, extending existing ones, with the run clock injected and no real wait:
- `TestStartup_ReadyProbeTiming` keeps the row "ready on the first probe never sleeps" and replaces the row "probe cadence matches Wait" with rows under the shipped cadence (`PollIntervalMS` 1000, `LivenessEveryNPolls` 10, `StartupTimeoutS` 60): ready on the second probe elapses exactly 1 s; an unchanged capture until ready on the seventh probe elapses 1+2+4+8+10+10 = 35 s, which pins both the doubling and the 10 s ceiling; and captures `a,a,b,b,b` before ready on the fifth probe elapse 1+2+1+2 = 6 s, which pins the reset on a change.
  The table gains a `captures []string` field that seeds `fakeReed.CaptureQueue`.
  The seventh-probe row leaves captures empty, so every capture reads `""` and its first sleep is the base only because the first probe always sleeps the base.
  Its comment and its `//testtiming:keep` reason are reworded for these rows.
- `TestStartup_TickCap` computes `wantTicks` from `pollInterval(cfg)` alone, and gains a subtest under a frozen clock with `LivenessEveryNPolls` 10 and `StartupTimeoutS` 5 that asserts the `Status` call count is `startupTickCap(5 s, 1 s)`; the old multiple-of-the-gap interval would give a smaller count, so the subtest fails if `maxTicks` is not computed from the base interval.
- `TestStartup_GateDismissalGap` is the one new top-level test, because no existing test drives probes closer together than the gap with a gate on screen; the shuttleengine row in `test-budget.yaml` has one slot free.
  Every row runs `StartGated` on a fake clock with `LivenessEveryNPolls` 10, `StartupTimeoutS` 25, a `StartupScript` of `StartupTrustPrompt` that never clears, and a generated `CaptureQueue` of alternating gate captures so every probe sees a changed capture and the sleep stays 1 s.
  Row one uses `fakeEngine`'s single-Enter sequence and asserts exactly three `Enter` key calls (the probes at 0 s, 10 s and 20 s), `errors.Is(err, ErrNotStarted)` and `run.json`'s outcome `died`.
  Row two sets `fakeReed.SendKeyErr` so the first key of every play fails, and asserts the key calls equal the number of `Status` calls, because a play whose first key failed never starts the gap.
  Row three uses `gateKeysEngine`, which returns a `Down` then an `Enter`, over `failingEnterReed`, whose `SendKey` delegates both keys to the embedded `fakeReed.SendKey` (recording each) and then returns an error for `Enter` alone, and asserts exactly six key calls, because a play that sent one key and failed partway starts the gap.
  Row four gives `gateKeysEngine` an empty sequence and asserts it was asked for a sequence on every probe (its own `plays` count equals the `Status` call count) with no key sent, because an empty sequence never starts the gap.
  `gateKeysEngine` therefore carries an `inputs []PaneInput` field returned as its sequence and a `plays int` counter incremented on each call.

**ImpactSummary:** `awaitStartup` has one caller (`start` in `run.go`), `checkLivenessTick` is shared by `awaitStartup` and `Wait` and `Wait`'s startup probe sits exactly one gap apart, so only `awaitStartup`'s cadence and the repeated-dismissal rule change; tests with `LivenessEveryNPolls` 1 see a ceiling equal to the base and an unchanged cadence, tests on `gateConfig` or `sparseProbeConfig` that go through `Start` are ready on the first probe and never sleep, and the shipped-cadence smoke tests only get faster.

**Commit:** 4: shuttle: probe startup at the poll interval with backoff and space gate dismissals
