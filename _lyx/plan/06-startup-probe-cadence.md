# Card 6 — startup-probe-cadence

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (int, error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalSpacing` -> `func TestStartup_GateDismissalSpacing(t *testing.T)`
- `plan:internal/shuttleengine#twoKeyGateEngine` -> `type twoKeyGateEngine struct`
- `plan:internal/shuttleengine#twoKeyGateEngine.TrustDismissSequence` -> `func (e *twoKeyGateEngine) TrustDismissSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#evenKeyFailingReed` -> `type evenKeyFailingReed struct`
- `plan:internal/shuttleengine#evenKeyFailingReed.SendKey` -> `func (r *evenKeyFailingReed) SendKey(guid, key string) error`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#fakeReed`

**Intent:** `awaitStartup` probes from a one-poll base with the daemon-wakeups backoff (D5 as superseded after Discussion), and a startup-gate dismissal is replayed at most once per dismissal gap (D6, with the after-Discussion rule that any play that sent a key starts the gap).
One card, because the faster probe without the spacing would let a capture taken before Claude redraws drive a second key into the next gate, the hazard D6 exists to keep closed.

`dismissalGap`, in `wait.go`, returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Wait` floors it: 10 s under the shipped template.
`Wait` is unchanged; its startup probe already runs once per gap.

`awaitStartup`'s pre-ready probe follows `PATTERN-daemon-wakeups`' backoff rule.
Its base is `pollInterval(cfg)` and its ceiling `dismissalGap(cfg)`.
Each iteration notes `run.lastStartupCapture` before calling `checkLivenessTick`; when the capture after the probe differs, the wait returns to the base.
It then sleeps the current wait on `run.clock` and doubles it, capped at the ceiling.
So a pane that keeps redrawing, as Claude's boot does, is probed every second, and a static screen or a failing capture backs off 1 s, 2 s, 4 s … up to the gap.
The tick cap is `startupTickCap(window, pollInterval(cfg))`, the count bound at the base cadence, so it still ends the loop under a clock that never advances; the startup deadline, the run deadline and every exit and its `allOutputFilesExist` consultation are unchanged, so `PATTERN-completion-signal` holds and `completionsignal_enforcement_test.go`'s pinned return counts do not move.
No sub-second literal is added, and `pattern/PATTERN-daemon-wakeups.md` is not amended.
`awaitStartup`'s "Probe cadence" doc paragraph is rewritten to this, one sentence per line, and `startupTickCap`'s comment says it takes the base poll interval rather than the probe interval.

`playInputsSent`, in `run.go`, is `playInputs`' loop returning how many inputs were delivered before the first failure, with the same error text.
`playInputs` keeps its signature and returns `playInputsSent`'s error, so its other callers are untouched.

`Run` gains a field `lastDismissAt time.Time`: the run clock's time of the last startup-gate play that sent at least one key, zero before any.
In `checkLivenessTick`'s `StartupTrustPrompt` branch, a probe whose `run.clock.Now()` is less than `dismissalGap(cfg)` past a non-zero `lastDismissAt` plays nothing, calls no `TrustDismissSequence`, and falls through to `classifyStartupWindow` as before; `checkLivenessTick` gains no return and no `allOutputFilesExist` call.
Otherwise it plays the engine's sequence through `playInputsSent` and, when at least one input was sent, sets `lastDismissAt` to that probe's time, whether the play then failed or not; an empty sequence or a play whose first key failed leaves `lastDismissAt` as it was, so the next probe may dismiss.
The existing Warn on a failed play and Info on a dismissal stay.
The first gate a run sees is dismissed on the probe that sees it, and a probe at or past one gap after the last play may dismiss, so `Wait`, whose probes are one gap apart, is never held.
Bound: the throttle only removes key presses and never adds one; a gate that never clears still ends at the startup deadline or the tick cap as died.

`template.yaml`'s `liveness_every_n_polls` comment gains: it also spaces startup-gate dismissals, at most one per `poll_interval_ms` times this value.
The line keeps its `liveness_every_n_polls: 10 ` prefix, which `internal/loomcli`'s integration test rewrites by exact string; no config key is added.
`internal/shuttleengine/doc.go`'s startup-probe paragraph ("Start/StartGated/Run/RunGated run the startup probe …") gains: the probe starts at one poll interval and backs off to the dismissal gap while the pane capture is unchanged, and a gate dismissal is replayed at most once per gap.

Tests, on the injected fake clock, never waiting out an interval (`PATTERN-test-speed`):
`TestStartup_ReadyProbeTiming` replaces its row "probe cadence matches Wait" with two rows and gains a per-row capture queue.
"a changing capture keeps the probe at one poll interval": config `PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60`, captures `boot-1`, `boot-2`, `ready`, script pending, pending, ready; want 3 status calls and exactly 2 s elapsed, which is the acceptance criterion's ready-on-the-first-probe-after-ready with one-poll sleeps.
"an unchanged capture backs off to the dismissal gap": `LivenessEveryNPolls: 4`, one repeating capture `boot`, script five pendings then ready; want 6 status calls and 1+2+4+4+4 = 15 s elapsed.
Its doc comment, and `startup_test.go`'s header clause on the cadence assertion, describe the backoff instead of the Wait-matching cadence.
`TestStartup_TickCap` computes its wanted tick count from `startupTickCap(timeout, pollInterval(cfg))`.
`TestStartup_GateDismissalSpacing` is new, table-driven, with config `PollIntervalMS: 1000, LivenessEveryNPolls: 3, StartupTimeoutS: 6`, a gate on every probe (script `StartupTrustPrompt`) and eight distinct gate captures so the backoff never engages and probes fall at 0 s … 7 s.
Its rows: "keys that land are spaced one gap apart" (`fakeEngine`'s one-Enter sequence, want 3 `SendKey` calls, at 0 s, 3 s and 6 s); "a play whose first key fails leaves the next probe free" (`fakeReed.SendKeyErr` set, want 8 calls, one per probe); "a play that fails after its first key still starts the gap" (`twoKeyGateEngine`, whose `TrustDismissSequence` answers two Enters, over `evenKeyFailingReed`, whose `SendKey` records through the embedded `fakeReed` and fails every second call, want 6 calls).
Every row asserts the start fails wrapping `ErrNotStarted` with run.json's outcome `died`, which is the gate-that-never-clears criterion.
The test and its subtests call `t.Parallel`: every row builds its own fakes, clock and fixture and asserts no logger output.
The two test types sit in `startup_test.go` beside `undismissableEngine` and `flakyStatusReed`, embedding `*fakeEngine` and `*fakeReed` the same way.
The new top-level test brings `internal/shuttleengine` to 161, its row in `cmd/lyx/testdata/test-budget.yaml`, so the budget is not raised.

**ImpactSummary:** Confined to `internal/shuttleengine`: every `Start`, `StartGated` and `RunGated` passes its startup probe about a second after the provider redraws ready instead of up to 10 s later, gate dismissals keep their 10 s spacing, `playInputs`' other callers and `Wait`'s cadence are unchanged, and one template comment changes.
