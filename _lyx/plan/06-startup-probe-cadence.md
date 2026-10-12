# Card 6 — Startup probes every poll interval, dismissing a gate once per gap

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#playInputsCounted` -> `func playInputsCounted(reed ReedOps, guid string, inputs []PaneInput) (sent int, err error)`
- `plan:internal/shuttleengine#sleepRecordingClock` -> `type sleepRecordingClock struct`
- `plan:internal/shuttleengine#secondKeyFailsReed` -> `type secondKeyFailsReed struct`
- `plan:internal/shuttleengine#keysDismissEngine` -> `type keysDismissEngine struct`
- `plan:internal/shuttleengine#TestStartup_DismissalGap` -> `func TestStartup_DismissalGap(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/loomcli#integrationDriverShuttleConfig`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Intent:** D5 and D6 with the parent's backoff amendment; they land together because probing every poll interval is safe only with the dismissal throttle, and the throttle is observable only at that cadence.

`dismissalGap(cfg)` is `pollInterval(cfg)` times `LivenessEveryNPolls` floored to 1 exactly as `Wait` floors it (10 s under the shipped template).
`Wait` keeps its own inline flooring; it is not refactored.

Throttle (D6).
`Run` gains a field `lastDismissAt time.Time`, zero until a play sent a key.
`checkLivenessTick`'s `StartupTrustPrompt` case plays `TrustDismissSequence` only when `lastDismissAt` is zero or `run.clock.Now().Sub(lastDismissAt) >= dismissalGap(cfg)`, so a probe at or past one gap after the last play may dismiss and the first gate a run sees is dismissed on the probe that sees it.
A probe inside the gap that still sees a gate plays nothing and falls through to `classifyStartupWindow` as before.
`playInputs` keeps its signature and delegates to the new `playInputsCounted`, which returns how many inputs were played before the first failure; the trust case calls `playInputsCounted` and, when `sent >= 1` (failed partway or not), sets `lastDismissAt` to the run clock's now.
An empty sequence (`sent == 0`, no error) and a play whose first key failed (`sent == 0`) leave `lastDismissAt` untouched, so the next probe may dismiss.
The logged "dismissed startup gate" line and the failure Warn keep their conditions.
The throttle only removes key presses and never adds one; the startup deadline and the tick cap still end a gate that never clears.
`Wait`'s startup probe runs at one gap with at least one gap between probes, so under the at-or-past rule the throttle never holds a dismissal there, and a started or attached run never reaches the case.

Cadence (D5, amended).
`awaitStartup` sleeps a per-probe interval instead of `pollInterval(cfg)` times `LivenessEveryNPolls`.
The interval starts at the base `pollInterval(cfg)` and, after each probe, doubles (capped at the ceiling `dismissalGap(cfg)`) when the pane capture the probe recorded in `run.lastStartupCapture` equals the previous probe's, and returns to the base when it differs; the first sleep is the base.
This is the `PATTERN-daemon-wakeups` backoff rule (base, doubling while nothing changes, a named ceiling, reset on a change); `PATTERN-daemon-wakeups` itself is not amended and no sub-second literal is added.
A ready provider shows within the first one or two probes because Claude's boot redraws the pane and resets the backoff.
`startupTickCap` is called with the base interval, so the tick cap is the largest the window could need and still bounds the loop under a clock that never advances; its signature and arithmetic are unchanged.
Each probe still runs `checkLivenessTick` (one reed status and, before ready, one pane capture), so `PATTERN-completion-signal` is untouched: every negative answer still comes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`.
The `awaitStartup` doc comment's "Probe cadence" paragraph is rewritten in place, one sentence per line, to describe the base interval, the backoff and the dismissal gap; the stale claim that probing at every poll is the stray-keypress hazard is replaced by the throttle that now prevents it.
What it lets through: probes (a status call and a pane capture each) run up to once a second where they ran every 10 s; they run once per start, stop at the first ready probe, never exceed the tick cap, and back off while the pane is static.

Edits outside the engine.
`template.yaml`'s comment on `liveness_every_n_polls` also says it spaces startup-gate dismissals; no key is added.
`integrationDriverShuttleConfig`'s doc comment is corrected: the startup step now probes every poll with a backoff ceiling of the dismissal gap, and the `liveness_every_n_polls: 1` override keeps that ceiling at the poll interval so the probes stay once per poll for the stub's short settle delay; the override line itself stays.
`doc.go`'s startup paragraph gains one sentence on the probe cadence and the once-per-gap dismissal.

Tests, on the fake clock, none waiting out an interval:
- `TestStartup_ReadyProbeTiming` becomes a table over recorded sleeps: its `cfg`-and-`wantElapsed` rows give way to `wantSleeps []time.Duration`, read from a `sleepRecordingClock` (embeds `*fakeClock`, appends each `Sleep`'s duration) and a per-row `captures` queue.
  Rows: ready on the first probe sleeps nothing; a pane whose capture changes every probe (ready on the third probe) sleeps `[1s, 1s]` under `PollIntervalMS` 1000 and `LivenessEveryNPolls` 10, the acceptance criterion "ready on the first probe after the pane shows ready, sleeps of one poll interval"; a static capture backs off `[1s, 2s, 4s, 8s, 10s, 10s]` to the ceiling; a capture that changes mid-way (`x x x y y` then ready) sleeps `[1s, 2s, 4s, 1s, 2s]`, the reset.
  Its `//testtiming:keep` reason and the doc comment above it, which say "exactly two probe intervals", are rewritten to the sleeps it now pins.
- `TestStartup_TickCap` computes `wantTicks` from `startupTickCap(timeout, pollInterval(cfg))`, no longer multiplying by `LivenessEveryNPolls`.
- `TestStartup_DismissalGap` is the one new top-level test (the `internal/shuttleengine` row in `cmd/lyx/testdata/test-budget.yaml` has one row of headroom over the present count, so the file is not edited), a table on `PollIntervalMS` 1000, `LivenessEveryNPolls` 10, `StartupTimeoutS` 25, with the pane capture different at every probe so the backoff stays at the base and probes fall one second apart; each row counts the keys the fake reed received and checks the start ends `ErrNotStarted` with persisted outcome died at the startup deadline:
  - the per-probe-distinct captures come from a `CaptureQueue` the test fills with one numbered gate screen per probe (more entries than the tick cap), since `fakeReed` repeats its last entry;
  - a gate that never clears gets 3 Enters, at t = 0, 10 and 20, and the start ends at the startup deadline rather than the tick cap: the fake clock has advanced exactly 26 s (the first probe past 25 s) and `Status` was called 27 times, fewer than `startupTickCap`'s 28;
  - a gate that first appears on the third probe is dismissed on that probe, then once per gap (3 Enters);
  - a reed whose `SendKeyErr` is set, so the first key fails, attempts a key on every probe (as many `SendKey` calls as `Status` calls), since a failed first key leaves the next probe free;
  - `keysDismissEngine` with `keys` 0 (embeds `*fakeEngine`, `TrustDismissSequence` returns that many Enters and counts its calls) is asked to dismiss on every probe (as many calls as `Status` calls) and sends no key, since an empty sequence leaves the next probe free;
  - `keysDismissEngine` with `keys` 2 over `secondKeyFailsReed` (embeds `*fakeReed`, its second `SendKey` call fails) sends 2 keys per play at t = 0, 10 and 20 (6 `SendKey` calls), since a play that sent one key and failed partway still starts the gap.
- `TestStartup_TrustPromptThenReady` and `TestStartup_UndismissableGateUntilWindowExpires` run under `fastConfig` (`LivenessEveryNPolls` 1, gap 1 s) and keep passing unchanged.

**ImpactSummary:** `awaitStartup` and `checkLivenessTick` serve every `Start`, `StartGated`, `Run` and `RunGated`, so every shuttle start now probes up to once a second; `Wait`'s own cadence, a re-attached run and the completion-signal paths are unchanged, and tests under `fastConfig` or `shortStartupConfig` see a ceiling equal to the base and so identical timing.
