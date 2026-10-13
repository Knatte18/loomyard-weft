# Card 4 — The startup probe runs every poll interval, with gate dismissals spaced by the dismissal gap

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsSent` -> `func playInputsSent(reed ReedOps, guid string, inputs []PaneInput) (sent bool, err error)`
- `plan:internal/shuttleengine#TestStartup_GateDismissalGap` -> `func TestStartup_GateDismissalGap(t *testing.T)`
- `plan:internal/shuttleengine#keyFailReed` -> `type keyFailReed struct`
- `plan:internal/shuttleengine#keyFailReed.SendKey` -> `func (r *keyFailReed) SendKey(guid, key string) error`
- `plan:internal/shuttleengine#sequenceDismissEngine` -> `type sequenceDismissEngine struct`
- `plan:internal/shuttleengine#sequenceDismissEngine.TrustDismissSequence` -> `func (e *sequenceDismissEngine) TrustDismissSequence(capture string) []PaneInput`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fastConfig`
- `internal/shuttleengine#newFakeClock`

**Intent:** Decisions D5 and D6 land together: probing every second without the throttle would replay a gate's dismissal on every probe, the stray-keypress hazard the 10-second cadence existed to prevent.

D5 — `awaitStartup` sleeps `pollInterval(cfg)` between probes instead of `pollInterval(cfg) × liveness_every_n_polls`, and its tick cap is `startupTickCap(window, pollInterval(cfg))`; it no longer reads `LivenessEveryNPolls`.
Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture; the exits, the deadline and the tick-cap answers are unchanged, so every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist` (`PATTERN-completion-signal`).
`awaitStartup`'s "Probe cadence" doc paragraph is rewritten, one sentence per line: it probes every poll interval because the provider is ready within about a second, and the dismissal gap, not the probe interval, spaces gate dismissals.
`startupTickCap`'s doc comment names the poll interval as the interval it is handed, in place of "the probe interval (not the raw poll interval)".
Wait's own probe cadence, and a re-attached run's, are unchanged.

D6 — `dismissalGap` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls` floored to 1 exactly as `Wait` floors it (10 s under the shipped template).
`Run` gains a field `lastDismissAt time.Time`, the run clock's time of the last dismissal play that sent a key, zero before any.
In `checkLivenessTick`'s `StartupTrustPrompt` case a dismissal is played only when `lastDismissAt` is zero or `run.clock.Now()` is at or past `lastDismissAt + dismissalGap(cfg)`; then `TrustDismissSequence(capture)` is played through `playInputsSent`, and `lastDismissAt` is set to the run clock's time whenever `sent` is true, whether or not a later input failed.
A probe inside the gap neither asks the engine for a sequence nor plays one, and falls through to `classifyStartupWindow` as today.
The warn on a failed play and the info on a successful one are kept.
`playInputsSent` is `playInputs`'s body reporting whether the first input was delivered: `sent` is false only for an empty sequence or a first input that failed; `playInputs` becomes `_, err := playInputsSent(…); return err`, so its other callers are unchanged.
Bound: the throttle only removes key presses, never adds one; the first gate a run sees is dismissed on the probe that sees it; the startup deadline and the tick cap still end a gate that never clears.
Wait's own startup probe runs at least one dismissal gap apart, so the at-or-past rule never holds a dismissal there.

`template.yaml`'s comment on `liveness_every_n_polls` adds that it also spaces startup-gate dismissals, at `poll_interval_ms` × this value; no config key is added.
`doc.go`'s closing startup paragraph adds that the startup probe runs every `poll_interval_ms` and replays a gate dismissal at most once per dismissal gap.
`pattern/PATTERN-daemon-wakeups.md`'s "What counts" section adds, after the sentence on bounded waits within a minute: a wait with a deadline of its own and a count cap is outside the rule whatever its length, bounded by that deadline and cap, and shuttle's startup probe (`awaitStartup`, bounded by `startup_timeout_s` and `startupTickCap`) is one; it gets no backoff.
No sub-second literal is added.

Tests (all on the fake clock, never waiting out an interval):
- `TestStartup_ReadyProbeTiming`: the `probe cadence matches Wait` row becomes `probes every poll interval whatever the liveness multiple`, same config (`PollIntervalMS: 1000, LivenessEveryNPolls: 10`) and script (pending, pending, ready), wanting 3 status calls and exactly `2 * time.Second` elapsed; the test's doc comment and its `//testtiming:keep` reason say one poll interval per probe; `startup_test.go`'s file comment names the probe-every-poll-interval and dismissal-gap assertions in place of "the probe-cadence-matches-Wait assertion".
- `TestStartup_TickCap`: `wantTicks` is computed as `startupTickCap(window, pollInterval(cfg))`.
- `TestStartup_GateDismissalGap` (new, table-driven; config `PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 25, RunTimeoutMin: 5`; a `fakeEngine` scripted `StartupTrustPrompt` for every probe, a live status and a gate capture, so the gate never clears and the start ends at the startup deadline as `OutcomeDied` wrapping `ErrNotStarted`).
  `keyFailReed` embeds `*fakeReed` and fails, after recording it, every `SendKey` of its `failKey`; `sequenceDismissEngine` embeds `*fakeEngine` and returns its `inputs` from `TrustDismissSequence`.
  Rows: `a gate that never clears gets one dismissal per gap` (plain Enter sequence, no failing key) wants exactly one Enter at virtual elapsed 0 s, 10 s and 20 s, 3 in all, fewer than the status calls; `a dismissal whose first key fails leaves the next probe free` (Enter fails) wants an Enter attempt on every probe, equal in count to the status calls; `a dismissal that fails after its first key still starts the gap` (`Down`, `Enter` with Enter failing) wants 3 `Down` and 3 `Enter` calls.
  Every row asserts the outcome is died and `errors.Is(err, ErrNotStarted)`.
  The test records the elapsed time of each `SendKey` by reading the fake clock in `keyFailReed.SendKey`.

**ImpactSummary:** `awaitStartup` probes up to ten times as often under the shipped template (one status and one capture per second until ready, never past `startupTickCap`); dismissals keep their 10 s spacing; `playInputs`'s six other callers see no change; Wait's cadence is untouched.

**Commit:** 4: The startup probe runs every poll interval, with gate dismissals spaced by the dismissal gap
