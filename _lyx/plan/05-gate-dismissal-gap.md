# Card 5 — gate-dismissal-gap

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (sent bool, err error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeReed.SendKey`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#fakeEngine.TrustDismissSequence`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Intent:** A startup-gate dismissal is replayed at most once per dismissal gap (decision D6 and its sent-a-key addendum), so the probe cadence can later rise without a second key reaching the next gate.
`dismissalGap(cfg)` is `pollInterval(cfg)` times `cfg.LivenessEveryNPolls` floored to 1 exactly as `Wait` floors it; it is 10 seconds under the shipped template.
`Run` gains a field `lastDismissAt time.Time`, zero until a play that sent a key.
`playInputsSent` is `playInputs`' loop returning whether at least one input was delivered, whether the play then failed partway or not; `playInputs` returns its error, so its other callers are unchanged.
In `checkLivenessTick`'s trust-prompt case the engine's sequence is played only when `lastDismissAt` is zero or `run.clock.Now().Sub(lastDismissAt) >= dismissalGap(cfg)`: a probe at or past one gap after the last play may dismiss, and the first gate a run sees is dismissed on the probe that sees it.
A play that sent at least one key sets `lastDismissAt` to the run clock's now; an empty sequence or a play whose first key failed leaves it unchanged, so the next probe is free to dismiss.
The success log line stays tied to a play that ended with no error and at least one input.
A probe inside the gap that still sees a gate plays nothing and falls through to `classifyStartupWindow` as before, so a gate that never clears still ends the start at `startup_timeout_s` as died.
Bound: the throttle only removes key presses and never adds one; `Wait`'s own startup probe runs at least one gap apart with the same floored multiple, so under the at-or-past rule it never holds a dismissal there, and `PATTERN-completion-signal` is unaffected because every negative answer still goes through `checkLivenessTick` and `classifyStartupWindow`.
`template.yaml`'s comment on `liveness_every_n_polls` says it also spaces startup-gate dismissals; no key is added.
`doc.go`'s startup paragraph gains a sentence that a dismissal replays at most once per `poll_interval_ms × liveness_every_n_polls`.
Test support: `fakeEngine` gains a field `TrustDismissInputs []PaneInput` that `TrustDismissSequence` returns instead of its single Enter when non-nil, and `fakeReed` gains a field `SendKeyFailAt int` so that, when above zero, `SendKey` returns `SendKeyErr` only on that one 1-based call and nil on every other (zero keeps today's behavior of returning `SendKeyErr` on every call).
The one new top-level test is within the package's row in `cmd/lyx/testdata/test-budget.yaml`.
`TestStartup_GateDismissalGap` starts a run on a ready fake engine with `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60, RunTimeoutMin: 5}`, then points the engine at a trust prompt and calls `checkLivenessTick` one virtual second apart, counting `SendKey` calls over 25 probes.
Its rows: a gate that stays up is dismissed on the first probe, the 11th and the 21st, three times and not once per probe, which also covers the at-or-past-one-gap boundary; a first key that fails (`SendKeyErr` set, `SendKeyFailAt` zero, so every call fails) leaves every probe free to dismiss, one attempted key per probe; a two-key sequence whose second key fails (`TrustDismissInputs` of two keys, `SendKeyFailAt` 2, `SendKeyErr` set) starts the gap after the first key was sent, so over the 25 probes `SendKey` is called six times, two on each of the same three probes.
A gate that never clears still ends the start at the startup deadline as died through `classifyStartupWindow`, which the throttle does not touch; `TestStartup_UndismissableGateUntilWindowExpires` already pins that exit for a trust-prompt classification, so this card adds no row for it.
The `awaitStartup` probe-cadence comment sentence that says `checkLivenessTick` replays the dismissal on every probe whose capture still shows a gate is reworded in this card, since the replay is now spaced by the gap; `checkLivenessTick`'s "at most once or twice per run" comment on the dismissal log line is reworded the same way.

**ImpactSummary:** `checkLivenessTick` is shared by `Wait` and `awaitStartup`, and `Wait`'s gap-spaced probes are never held; `playInputs` keeps its signature and its four other callers, and the two fakes gain fields that default to today's behavior.
