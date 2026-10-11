# Card 4 — The startup probe backs off from the poll interval and spaces gate dismissals

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (sent int, err error)`
- `plan:internal/shuttleengine#keyFailReed` -> `type keyFailReed struct`
- `plan:internal/shuttleengine#keyFailReed.SendKey` -> `func (r *keyFailReed) SendKey(guid, key string) error`
- `plan:internal/shuttleengine#scriptedDismissEngine` -> `type scriptedDismissEngine struct`
- `plan:internal/shuttleengine#scriptedDismissEngine.TrustDismissSequence` -> `func (e *scriptedDismissEngine) TrustDismissSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#TestStartup_DismissalGap` -> `func TestStartup_DismissalGap(t *testing.T)`

**Edit:**
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.classifyStartupWindow`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#fakeClock`

**Intent:**
D5 as superseded by the parent's backoff addition, D6, and the parent's addition that any play that sent a key starts the dismissal gap.

`dismissalGap` in `wait.go` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Wait` floors it: 10 s under the shipped template.
It is the backoff's named ceiling and the dismissal gap both.

Backoff.
`awaitStartup` probes from a base of `pollInterval(cfg)` instead of sleeping `pollInterval(cfg) × liveness_every_n_polls` between probes.
After each probe it compares `run.lastStartupCapture` with the value it held before that probe: the first probe, and any probe whose capture differs, set the next sleep to the base; a probe whose capture is unchanged, a failed capture or a status error included, doubles the previous sleep, capped at `dismissalGap(cfg)`.
The tick cap is `startupTickCap(window, pollInterval(cfg))`, the most probes the window can hold at the base, so the count bound stays sound however the sleeps grow.
Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture.
This follows `PATTERN-daemon-wakeups`' backoff rule (a base, doubling while nothing changes, a named ceiling, reset on a change), with no sub-second literal added to `wait.go`, a file the guard scans; `PATTERN-daemon-wakeups.md` is not amended.
A ready provider still shows within the first one or two probes because Claude's boot redraws the pane and resets the backoff.
Under `LivenessEveryNPolls` 1 the ceiling equals the base, so a start under `fastConfig` or `shortStartupConfig` keeps today's cadence.
The fixture configs with a sparser liveness cadence, `sparseProbeConfig` and `gateConfig`, reach `awaitStartup` only with an engine ready on its first probe, which never sleeps, or drive `Wait` directly, so their timings are unchanged too.
`awaitStartup`'s "Probe cadence" doc paragraph is rewritten to state the base, the doubling, the reset on a changed capture, the ceiling and why the dismissal throttle, not the probe cadence, now keeps a second key out of the next gate; one sentence per line (`PATTERN-comment-line-breaks`).
`startupTickCap`'s doc comment says its interval is the base probe interval.

Dismissal gap.
`Run` gains a field `lastDismissAt time.Time`, zero until a dismissal sends a key.
`playInputsSent` in `run.go` is `playInputs`' loop returning how many inputs it played before it returned; `playInputs` calls it and drops the count, so its callers are unchanged.
On `StartupTrustPrompt`, `checkLivenessTick` plays the engine's trust-dismiss sequence through `playInputsSent` only when `lastDismissAt` is zero or `run.clock.Now()` is at or past `lastDismissAt` plus `dismissalGap(cfg)`.
A play that sent at least one key, failed partway or not, sets `lastDismissAt` to the time the play started; an empty sequence, or a play whose first key failed, leaves it, so the next probe may dismiss.
A probe inside the gap that still sees a gate plays nothing, logs at Debug that the dismissal is held, and classifies the startup window as before; the first gate a run sees is dismissed on the probe that sees it.
The existing Warn on a failed play and Info on a played dismissal stay.
What the throttle lets through and its bound: it only removes key presses and never adds one, and the startup deadline and tick cap still end a gate that never clears.
`Wait` keeps its own cadence: its probe already runs at least one dismissal gap apart on the run's clock, so under the at-or-past rule the throttle never holds a dismissal there.
`template.yaml`'s comment on `liveness_every_n_polls` adds that it also spaces startup-gate dismissals; no config key is added.
Every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist` (`PATTERN-completion-signal`).
This card adds no exit, no `fmt.Errorf` call and no `Outcome` token to `awaitStartup` or `checkLivenessTick`, so the per-function counts `completionsignal_enforcement_test.go` pins stay as they are.

Tests.
`TestStartup_ReadyProbeTiming` gains a `captures []string` column fed to the reed fake's `CaptureQueue`, and its "probe cadence matches Wait" row is replaced by two rows under `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60, RunTimeoutMin: 5}`.
"a changing capture keeps the probe at the poll interval": script Pending, Pending, Ready with captures `a`, `b`, `c`; 3 status calls and exactly 2 s elapsed, so ready is reached on the first probe after the pane shows ready with sleeps of one poll interval.
"an unchanged capture doubles the probe up to the dismissal gap": script five Pending then Ready with one capture throughout; 6 status calls and exactly 1+2+4+8+10 = 25 s elapsed.
The first row ("ready on the first probe never sleeps") stays, and the test's doc comment and `//testtiming:keep` reason are reworded from "exactly two probe intervals" to the base-plus-backoff timing.
The header comment of `startup_test.go`, which lists "the probe-cadence-matches-Wait assertion", names the base-plus-backoff timing rows and the dismissal gap instead.
`TestStartup_TickCap`'s `wantTicks` computes `startupTickCap` from `pollInterval(cfg)`, the interval `awaitStartup` now passes.

`TestStartup_DismissalGap` is a new table test in `startup_test.go`, each row a start under `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 20, RunTimeoutMin: 5}`, a `fakeClock`, a `StartupScript` of `StartupTrustPrompt` alone (it sticks), and a `CaptureQueue` of 40 distinct captures.
Every probe then sees a changed capture, so the probes run one second apart, at 0 s through 20 s, and the probe at 21 s, past the deadline, classifies the start died.
Every row asserts `StartGated` returns an error wrapping `ErrNotStarted` and the run record reads died, so a gate that never clears still ends at the startup deadline as died.
- "a gate on consecutive one-second probes is dismissed once per gap": plain `fakeEngine` and `fakeReed`; exactly 3 `SendKey` Enter calls (at 0 s, 10 s and 20 s), and the first `SendKey:Enter` in `CallLog` comes before the second `Status`.
  A throttle that dismissed only strictly past the gap would send 2 (at 0 s and 11 s), and today's code sends one per probe.
- "an empty sequence leaves the next probe free": `scriptedDismissEngine` answering an empty sequence first and `Enter` after; exactly 3 Enter calls (at 1 s, 11 s and 21 s), the first between the second and third `Status` in `CallLog`.
  Starting the gap on the empty play would delay the first Enter to 10 s and send 2.
- "a play whose first key fails leaves the next probe free": `fakeReed.SendKeyErr` set; the `SendKey` call count equals the `Status` call count, one attempt per probe.
  Today's code passes it too; the row pins the bound that a failed first key starts no gap.
- "a play that fails after sending a key starts the gap": `scriptedDismissEngine` answering `Down` then `Enter`, and `keyFailReed` failing `Enter`; exactly 3 `Down` and 3 `Enter` calls, where today's code sends one of each per probe.
`keyFailReed` embeds `*fakeReed`, records the call through the embedded `SendKey` and then returns a scripted error when the key equals its `failKey` field.
`scriptedDismissEngine` embeds `*fakeEngine` and answers `TrustDismissSequence` from a `sequences [][]PaneInput` field, consumed in order with the last one sticking.

`TestRun_Wait_StartupWindow` gains a `cfg Config` column, `shortStartupConfig` when zero, and a `wantEnters int` column, checked when non-zero.
Its new row "a gate probed once per dismissal gap in Wait is dismissed on every probe" runs `Wait` on a not-started run under `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 25, RunTimeoutMin: 5}` with `StartupTrustPrompt` alone and the gate capture, and wants `OutcomeDied` and 3 Enter calls.
`Wait`'s liveness probes fall at 9 s, 19 s and 29 s, each exactly one gap after the last, and the last classifies the expired window after its dismissal.
A throttle that held a dismissal at exactly one gap would send 2, so the row pins that the throttle never holds a dismissal in `Wait`.
The new top-level test brings `internal/shuttleengine` to its row in `cmd/lyx/testdata/test-budget.yaml`, which already budgets one more test than the package holds.

**ImpactSummary:** Confined to the startup step in `internal/shuttleengine`: `awaitStartup`'s sleeps and tick cap, a run-clock throttle on the dismissal both `awaitStartup` and `Wait` reach through `checkLivenessTick`, and a counting variant of `playInputs` whose other callers are untouched.

**Commit:** 4: The startup probe backs off from the poll interval and spaces gate dismissals
