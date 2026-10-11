# Card 4 — `awaitStartup` probes from the poll interval with backoff, and spaces gate dismissals

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/websterengine#AcquireStateMutation`
- `internal/loomcli#integrationDriverShuttleConfig`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`

**Delete:**
- `internal/shuttleengine#TestStartup_UndismissableGateUntilWindowExpires`

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsCounted` -> `func playInputsCounted(reed ReedOps, guid string, inputs []PaneInput) (int, error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#twoKeyGateEngine` -> `type twoKeyGateEngine struct`
- `plan:internal/shuttleengine#twoKeyGateEngine.TrustDismissSequence` -> `func (e *twoKeyGateEngine) TrustDismissSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#failingKeyReed` -> `type failingKeyReed struct`
- `plan:internal/shuttleengine#failingKeyReed.SendKey` -> `func (r *failingKeyReed) SendKey(guid, key string) error`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#undismissableEngine`

**Intent:** Decision D5 as the parent's addendum supersedes it (backoff under `PATTERN-daemon-wakeups`, no amendment to that background), and decision D6 with the addendum that any play that sent a key starts the gap.

`dismissalGap(cfg)` (in `wait.go`) returns `pollInterval(cfg)` times `LivenessEveryNPolls`, the multiple floored to 1 exactly as `Run.Wait` floors it: 10 s under the shipped template.

Probe cadence.
`Run.awaitStartup` starts its interval at `pollInterval(cfg)` and computes its tick cap as `startupTickCap(window, pollInterval(cfg))`.
The sleep after the first probe is the base interval.
After each later probe the next sleep returns to the base when `run.lastStartupCapture` differs from its value before that probe, and otherwise doubles, capped at `dismissalGap(cfg)`.
A failed capture leaves `lastStartupCapture` unchanged, so it counts as no change.
Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture; the loop stops at the first ready probe, and the startup deadline and the tick cap still end it.
The backoff is the `PATTERN-daemon-wakeups` rule: base `poll_interval_ms` at the one-second floor, a named ceiling, a reset on a change; no sub-second literal is added.
`awaitStartup`'s "Probe cadence" doc paragraph is rewritten to say this, and `startupTickCap`'s doc comment says its interval is the base poll interval, which bounds the count from above under any backoff.

Dismissal gap.
`Run` gains a field `lastDismissAt time.Time`, the run clock's time of the last startup-gate play that sent at least one key, zero before any.
`playInputsCounted` plays inputs exactly as `playInputs` does and also returns how many inputs were delivered before the first failure; `playInputs` becomes a call to it that drops the count, so its callers are unchanged.
`Run.checkLivenessTick`'s `StartupTrustPrompt` branch plays the engine's trust-dismiss sequence only when `lastDismissAt` is zero or the run clock reads at or past `lastDismissAt` plus `dismissalGap(cfg)`.
It plays through `playInputsCounted` and sets `lastDismissAt` to the clock's time read before the play whenever the count is at least one, a play that failed after its first key included.
An empty sequence or a play whose first key failed leaves `lastDismissAt` as it was, so the next probe may dismiss.
A probe inside the gap plays nothing and classifies the startup window as before.
Bound: the throttle only removes key presses, never adds one, and the startup deadline and tick cap still end a gate that never clears.
`Run.Wait`'s own startup probe runs at the dismissal gap already, so under the at-or-past rule the throttle never holds a dismissal there.
`template.yaml`'s comment on `liveness_every_n_polls` adds that its multiple of `poll_interval_ms` also spaces startup-gate dismissals.

Forced comment edits.
`AcquireStateMutation`'s doc comment replaces "typically one or two probe intervals, about 5–10 s under the shipped config" with the provider's own readiness time, about one or two seconds.
`integrationDriverShuttleConfig`'s doc comment drops the claim that the startup step sleeps a whole probe interval, and its stale `500ms` and `every tenth (5s)` figures; its `liveness_every_n_polls: 1` override stays, now keeping the backoff ceiling at one poll interval.
`startup_test.go`'s file header comment replaces "the probe-cadence-matches-Wait assertion" with the probe backoff and the dismissal gap, and its "undismissable-gate refusal" with the gate-dismissal table.

Tests.
`TestStartup_ReadyProbeTiming` gains a `captures []string` field fed to the reed's `CaptureQueue`, and its `//testtiming:keep` reason and doc comment are reworded to the backoff.
With `PollIntervalMS` 1000 and `LivenessEveryNPolls` 10 its rows are:
"ready on the first probe never sleeps" (kept);
"an unchanged pane backs off from the poll interval" (script pending, pending, ready, no captures; 3 status calls; 3 s, which replaces "probe cadence matches Wait");
"the backoff stops at the dismissal gap" (six pending then ready, no captures; 7 status calls; 1+2+4+8+10+10 = 35 s);
"a changed capture resets the backoff and a changing pane is probed every poll interval" (captures `a`, `a`, `a`, `b`, `c`, `d`; five pending then ready; 6 status calls; 1+2+4+1+1 = 9 s), which pins both the reset after a backed-off sleep and the acceptance criterion's one-interval sleeps while the pane changes.
`TestStartup_TickCap` computes its expected count as `startupTickCap(timeout, pollInterval(cfg))`.

`TestStartup_GateDismissalGap` is table-driven over a config of `PollIntervalMS` 1000, `LivenessEveryNPolls` 10 (a row may override it), `StartupTimeoutS` 20 and `RunTimeoutMin` 5, a fake clock, a live strand, a capture queue of distinct `gate <n>` strings longer than the probe count so every probe sees a changed pane one second after the last, and a startup script stuck at `StartupTrustPrompt`.
Probes run at 0 s to 21 s, and the probe at 21 s finds the window expired.
Every row asserts `StartGated` fails wrapping `ErrNotStarted`, the strand removed and the record died, and counts `SendKey` calls:
"a gate on one-second probes is dismissed once per gap" (fake engine, one Enter per play; plays at 0, 10 and 20 s; 3 calls);
"a play whose first key fails leaves the next probe free" (`failingKeyReed` failing `Enter`; a play on all 22 probes; 22 calls);
"a play that fails after its first key still starts the gap" (`twoKeyGateEngine` returning `Down` then `Enter`, `failingKeyReed` failing `Enter`; plays at 0, 10 and 20 s; 6 calls);
"a zero multiple floors the gap to one poll interval" (`LivenessEveryNPolls` 0; a play on all 22 probes; 22 calls);
"an empty dismissal sequence sends no key" (`undismissableEngine`; 0 calls, and the log holds no `shuttle: dismissed startup gate` line), which replaces `TestStartup_UndismissableGateUntilWindowExpires`, deleted here.
`TestRun_Wait_StartupWindow`'s row "trust prompt that never clears" additionally asserts one `Enter` per `Status` call, so a throttle that held Wait's at-gap probes fails it.
`twoKeyGateEngine` embeds `*fakeEngine` and overrides `TrustDismissSequence`.
`failingKeyReed` embeds `*fakeReed`, records every key through the embedded `SendKey`, and returns an error for its named key.
The new top-level test replaces the deleted one, so the `internal/shuttleengine` count in `cmd/lyx/testdata/test-budget.yaml` is unchanged.

**ImpactSummary:** The startup probe's cadence and the gate-key replay change inside `wait.go` only; `playInputs` keeps its signature for its other callers, Wait's cadence is unchanged, and two comments outside the package are corrected.

**Commit:** 4: Probe startup from the poll interval with backoff and space gate dismissals
