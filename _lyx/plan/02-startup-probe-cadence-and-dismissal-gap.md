# Card 2 — startup-probe-cadence-and-dismissal-gap

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsCounted` -> `func playInputsCounted(reed ReedOps, guid string, inputs []PaneInput) (sent int, err error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#flakyKeyReed` -> `type flakyKeyReed struct`
- `plan:internal/shuttleengine#flakyKeyReed.SendKey` -> `func (r *flakyKeyReed) SendKey(guid, key string) error`

**Uses:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#fakeReed.CapturePane`
- `internal/shuttleengine#fakeEngine.TrustDismissSequence`

**Intent:** D5 as amended by the parent's addition, and D6, in one commit so no commit probes at one second without the dismissal throttle.

Cadence.
`awaitStartup` sleeps a backoff between probes instead of `pollInterval(cfg) × liveness_every_n_polls`.
The sleep starts at `pollInterval(cfg)`.
After each probe, when the pane capture equals the previous probe's capture the next sleep doubles, capped at `dismissalGap(cfg)`;
when the capture differs the sleep returns to `pollInterval(cfg)`;
a failed capture leaves the sleep as it is.
Every probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture.
`maxTicks` is `startupTickCap(window, pollInterval(cfg))`, the base interval, which stays an upper bound on the probes the window admits since no sleep is shorter than the base;
`startupTickCap`'s doc comment drops "the probe interval (not the raw poll interval)" and names the base interval.
`awaitStartup`'s "Probe cadence" paragraph is rewritten, one sentence per line, to state the backoff, its ceiling, its reset, and that the stray-keypress bound now rests on the dismissal gap below rather than on the probe cadence;
the Wait-cadence reasoning it replaces is deleted.
`PATTERN-daemon-wakeups` is not amended: `awaitStartup` has a deadline and a tick cap, follows the background's backoff rule, and adds no sub-second literal, so `cmd/lyx/daemonwakeups_test.go`'s scan of `wait.go` stays green.
This card does not change `Wait`'s own probe cadence, which stays at `liveness_every_n_polls` polls.

Dismissal gap.
`dismissalGap(cfg)` returns `pollInterval(cfg) × LivenessEveryNPolls` with the multiple floored to 1 exactly as `Wait` floors it, 10 s under the shipped template.
`Run` gains an unexported field `lastGateDismissAt time.Time`, zero until a startup-gate play sends a key.
`checkLivenessTick`'s `StartupTrustPrompt` branch plays the engine's trust-dismiss sequence only when `lastGateDismissAt` is zero or the run clock's now is at or past `lastGateDismissAt + dismissalGap(cfg)`;
a probe inside the gap that still sees a gate plays nothing, logs nothing above Debug, and classifies the startup window as before.
The play goes through `playInputsCounted`, which is `playInputs`'s body returning how many inputs went out before the first failure, with `playInputs` reduced to a call of it that drops the count, so its other callers are untouched.
A play that sent at least one input, failed partway or not, sets `lastGateDismissAt` to the clock reading taken before the play;
an empty sequence or a play whose first input failed leaves it unchanged, so the next probe may dismiss.
The existing Warn on a failed play and the Info on a completed one stay.
The gate's floored multiple is read on every tick from the run's config, so no config key is added;
`template.yaml`'s comment on `liveness_every_n_polls` gains that the product with `poll_interval_ms` is also the shortest gap between two startup-gate dismissals.
`doc.go`'s paragraph on the startup probe gains two sentences: the probe runs every `poll_interval_ms`, backing off while the pane does not change up to `poll_interval_ms × liveness_every_n_polls`, and that product is the shortest gap between two dismissals of a startup gate.

Bound: the throttle only removes key presses and never adds one;
the startup deadline and the tick cap still end a gate that never clears, as died through `classifyStartupWindow`;
the per-second status checks and captures run once per start, stop at the first ready probe and never exceed the tick cap.
`Wait`'s startup probe already runs at the dismissal gap with at least one gap between probes, so under the at-or-past rule it is never held;
its code is not changed.
The input that reaches the throttled branch after every upstream check passes is a probe whose capture classifies `StartupTrustPrompt` while the clock is inside one gap after the last played dismissal;
`TestStartup_GateDismissalGap`'s first row reaches it.
Every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`, so `completionsignal_enforcement_test.go` needs no change.

Tests, on the fake clock, never waiting out an interval.
`TestStartup_ReadyProbeTiming` replaces its "probe cadence matches Wait" row with three rows under `PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60`:
"an unchanged pane doubles the sleep" (script pending, pending, ready; the fake reed's captures are all empty so they never change; three status calls, elapsed 3 s),
"a changing pane keeps the base interval" (a capture queue of distinct strings, script pending ×3 then ready; four status calls, elapsed 3 s),
and "the backoff stops at the dismissal gap" (unchanged captures, script pending ×6 then ready; seven status calls, elapsed 35 s, the sum of 1, 2, 4, 8, 10 and 10 s).
Its doc comment is rewritten for the new cadence.
`TestStartup_GateDismissalGap` is new, table-driven, with `flakyKeyReed`, a double like `flakyStatusReed` that fails its first `failFirst` SendKey calls before delegating to the embedded fake reed and records the fake clock's elapsed time at every call, which is how the rows read when each Enter went out:
"consecutive one-second probes get one dismissal per gap" (same config, a capture queue of distinct gate strings so no backoff, script trust prompt ×21 then ready; exactly three Enter calls, at 0, 10 and 20 s on the fake clock, and a run returned),
"a play whose first key fails leaves the next probe free" (`failFirst` 1; Enter attempted at 0 s, sent at 1 s, then at 11 s),
and "a gate that never clears ends at the startup deadline as died" (`StartupTimeoutS: 25`, script trust prompt only; the error wraps `ErrNotStarted`, the record reads died, the strand is removed, and Enter went out three times, at 0, 10 and 20 s).

**ImpactSummary:** `awaitStartup`'s sleep and tick cap, `checkLivenessTick`'s gate branch and one new `Run` field; `Wait` shares the throttle through `checkLivenessTick` and is never held by it, and every caller of `playInputs` sees the same signature.
