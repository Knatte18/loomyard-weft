# Card 4 — `awaitStartup` probes from the poll interval with backoff, and spaces gate dismissals

**Create:**
- `plan:internal/shuttleengine#dismissalGap` -> `func dismissalGap(cfg Config) time.Duration`
- `plan:internal/shuttleengine#dismissScriptEngine` -> `type dismissScriptEngine struct`
- `plan:internal/shuttleengine#dismissScriptEngine.TrustDismissSequence` -> `func (e *dismissScriptEngine) TrustDismissSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#failKeyReed` -> `type failKeyReed struct`
- `plan:internal/shuttleengine#failKeyReed.SendKey` -> `func (r *failKeyReed) SendKey(guid, key string) error`
- `plan:internal/shuttleengine#TestStartup_DismissalGap` -> `func TestStartup_DismissalGap(t *testing.T)`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#TestStartup_ReadyProbeTiming`
- `internal/shuttleengine#TestStartup_TickCap`
- `internal/shuttleengine#TestRun_Wait_StartupWindow`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/template.yaml`
- `internal/loomcli#integrationDriverShuttleConfig`

**Delete:**
- `internal/shuttleengine#TestStartup_UndismissableGateUntilWindowExpires`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#undismissableEngine`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`

**Intent:**
This card carries D5 as the parent's later decision replaced it (`awaitStartup` backs off under `PATTERN-daemon-wakeups`), and D6 together with the later decision that any play that sent a key starts the dismissal gap.
Every shuttle start sleeps `poll_interval_ms × liveness_every_n_polls` (10 s under the shipped template) between startup probes, so a start takes about 11 s although the provider is ready within about a second.
The long sleep existed only so a capture taken before Claude redraws after an Enter cannot drive a second key into the next gate.
This card moves that spacing from the probes to the dismissals.

`dismissalGap(cfg)` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, with the multiple floored to 1 exactly as `Wait` floors it.
That is 10 s under the shipped template.

`awaitStartup`'s probe cadence:
- The base sleep is `pollInterval(cfg)` and the ceiling is `dismissalGap(cfg)`.
- After each probe, the sleep is the base when this is the first probe or when `run.lastStartupCapture` differs from its value after the previous probe.
  Otherwise the sleep doubles from the previous one, capped at the ceiling.
  A probe that took no capture (a capture error, a status error, or an untracked strand) leaves `lastStartupCapture` unchanged, so it counts as no change.
- This is the backoff rule of `PATTERN-daemon-wakeups` as written: a base, doubling while nothing changes up to a named ceiling, and back to the base on a change.
  The background is not amended, and no sub-second literal is added to `wait.go`, which `cmd/lyx/daemonwakeups_test.go` scans.
  A booting Claude redraws its pane, which resets the backoff, so a ready provider still shows within one or two probes.
- `maxTicks` is `startupTickCap(window, pollInterval(cfg))`, an upper bound on the count of the backed-off probes.
- Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture.
  Every negative answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist` (`PATTERN-completion-signal`).
- Rewrite the doc comment's "Probe cadence" paragraph to state this cadence, and that `checkLivenessTick` spaces the gate dismissals, one sentence per line.
- Update `startupTickCap`'s doc comment: its interval is the base poll interval, not a probe interval.

`checkLivenessTick`'s `StartupTrustPrompt` branch now throttles the dismissal:
- `Run` gains a field `lastDismissAt time.Time`, the run clock's time read before the last dismissal play that sent a key.
  It is zero until the first such play.
- When `lastDismissAt` is not zero and `run.clock.Now()` is less than `dismissalGap(cfg)` after it, the branch asks for no sequence, plays nothing and logs one Debug line.
  It then classifies the startup window as before.
  A probe at or past one gap after the last play may dismiss, and the first gate a run sees is dismissed on the probe that sees it.
- Otherwise the branch takes `TrustDismissSequence(capture)`.
  - An empty sequence plays nothing and leaves `lastDismissAt` unchanged.
  - Otherwise it plays the first input alone through `playInputs`.
    If that fails, it logs the existing Warn and leaves `lastDismissAt` unchanged, so the next probe may dismiss.
  - Once the first input is sent, it sets `lastDismissAt` to the time read before the play and plays the rest through `playInputs`.
    A failure there logs the same Warn and keeps the gap, since a partly failed play can leave keys in the pane.
    A full play logs the existing Info line.
- Bound: the throttle only removes key presses and never adds one.
  A gate that never clears still ends at the startup deadline through `classifyStartupWindow`, or at the tick cap.
- `Wait`'s own startup probe runs every `livenessEvery` ticks of one `pollInterval` sleep each, which is at least one gap between probes.
  Under the at-or-past rule the throttle therefore never holds a dismissal there, and `Wait`'s cadence is unchanged.
- `checkLivenessTick`'s doc comment gains a sentence saying that a startup-gate dismissal plays at most once per `dismissalGap`, timed from the last play that sent a key.
  The "at most once or twice per run" remark beside the Info line is reworded so that a gate shown for a whole window can be dismissed once per gap.

`internal/shuttleengine/template.yaml`: the `liveness_every_n_polls` comment gains a clause saying that `poll_interval_ms` times this value is also the least time between two startup-gate dismissals and the longest startup-probe backoff.
No config key is added.

`internal/shuttleengine/doc.go`: the startup-probe paragraph gains one sentence.
The probe runs every `poll_interval_ms`, backs off to the dismissal gap while the pane does not change, and plays a gate dismissal at most once per gap.

`internal/loomcli#integrationDriverShuttleConfig`: its doc comment says the `liveness_every_n_polls: 1` override exists because the startup step "sleeps a whole probe interval", which this card makes false.
Reword it: the override sets the backoff ceiling to the base poll interval, so the startup probe never backs off past one poll interval while the stub's pane stays static before its ready marker, and Start returns within the stub's settle delay.
The override itself stays.

Tests (fake clock, never a real wait; `PATTERN-test-speed`):
- `TestStartup_ReadyProbeTiming`: the table gains a `captures []string` field, fed to the fake reed's `CaptureQueue`.
  The rows run on `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 60, RunTimeoutMin: 5}`.
  The "probe cadence matches Wait" row is replaced by three rows:
  - "a changing pane is probed every poll interval": script pending, pending, ready, with captures `a`, `b`, `c`.
    It makes 3 status calls in 2 s elapsed, so it reaches ready on the first probe after the pane shows ready, with one-poll-interval sleeps.
  - "a static pane backs off to the dismissal gap": six pending probes, then ready, with the one capture `booting`.
    The sleeps are 1, 2, 4, 8, 10 and 10 s, so it makes 7 status calls in 35 s.
  - "a pane change resets the backoff": five pending probes, then ready, with captures `A`, `A`, `A`, `B`.
    The sleeps are 1, 2, 4, 1 and 2 s, so it makes 6 status calls in 10 s.
  Rewrite the doc comment and the `//testtiming:keep` reason to name these timings.
  Rewrite the `startup_test.go` header comment's "probe-cadence-matches-Wait assertion" to name the probe backoff and the dismissal gap.
- `TestStartup_TickCap`: `cfg` becomes `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 30, RunTimeoutMin: 5}`.
  The expected count becomes `startupTickCap(30*time.Second, pollInterval(cfg))`, which is 33 and differs from the 6 the old gap-based interval would give, so the row pins the cap's interval.
- New `TestStartup_DismissalGap` in `startup_test.go`, table-driven over `StartGated`.
  It absorbs `TestStartup_UndismissableGateUntilWindowExpires`, which this card deletes, as its empty-sequence row, so the package keeps one test for a gate that never clears.
  - `dismissScriptEngine` embeds `*fakeEngine` and answers `TrustDismissSequence` from a FIFO list of sequences whose last entry repeats.
  - `failKeyReed` embeds `*fakeReed`: its `SendKey` records the call through the embedded fake, then returns an error for one named key.
  - Every row runs on `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 30, RunTimeoutMin: 5}`.
    The startup script is `StartupTrustPrompt` throughout.
    The captures are `gate 0` to `gate 40`, all distinct, so the backoff stays at one second and the pane shows a gate on consecutive one-second probes.
    The probes then run at 0 s to 31 s, and the probe at 31 s plays before the window closes.
  - Every row asserts:
    - that `StartGated` fails wrapping `ErrNotStarted`;
    - that `run.json`'s Outcome is died;
    - that `RemoveStrand` is called exactly once;
    - that the virtual elapsed time is 31 s;
    - that the log contains `shuttle: dismissed startup gate` exactly when the row expects a full play.
    That pins that a gate which never clears still ends the start at the startup deadline as died.
  - The rows, with the `SendKey` calls each expects:
    - "a gate on every one-second probe is dismissed once per gap": the sequence is `[Enter]`; there are 4 `Enter` calls, at 0, 10, 20 and 30 s, and the log has the dismissal line.
      A strict past-the-gap rule would play at 0, 11 and 22 s and give 3, so the row pins the at-or-past edge `Wait` relies on.
    - "a play whose first key fails leaves the next probe free": the sequence is `[Enter]`, and the `fakeReed`'s `SendKeyErr` fails every key; there are 32 `Enter` calls, one per probe, and no dismissal line.
    - "a play that fails after its first key starts the gap": the sequence is `[Down, Enter]` over a `failKeyReed` failing `Enter`; there are 8 calls, `Down` and `Enter` at 0, 10, 20 and 30 s, and no dismissal line.
    - "an empty sequence leaves the next probe free": the sequences are `[]`, then `[Enter]`; there are 4 `Enter` calls, at 1, 11, 21 and 31 s, and the log has the dismissal line.
      An empty play that started the gap would play at 10, 20 and 30 s and give 3.
    - "a gate whose accepting option cannot be located sends no key": this row is built on `undismissableEngine` and is moved from the deleted test; there are no `SendKey` calls and no dismissal line.
  - With one test added and one deleted, `internal/shuttleengine` stays at 160 top-level tests against its row of 161 in `cmd/lyx/testdata/test-budget.yaml`.
- `TestRun_Wait_StartupWindow` gains a per-row `cfg Config` field, where zero means `shortStartupConfig`, and a `wantEnters int` field.
  It gains the row "a gate that never clears is dismissed on every Wait probe".
  That row runs on `Config{PollIntervalMS: 1000, LivenessEveryNPolls: 10, StartupTimeoutS: 25, RunTimeoutMin: 5}` with the script `StartupTrustPrompt` throughout.
  `Wait` probes at its 10th, 20th and 30th ticks, at 9, 19 and 29 s, each one gap after the last.
  So the row expects 3 `Enter` calls and `OutcomeDied`.
  A throttle that held `Wait`'s probe at exactly one gap would skip the 19 s play and give 2.

**ImpactSummary:** `awaitStartup`, reached only from `start`, gets a new sleep and tick cap. `checkLivenessTick`'s gate branch, reached from `awaitStartup` and from `Wait`'s attach-time probe, gets the throttle, which never meets `Wait`'s probes since they run at least one gap apart. One `loomcli` test comment's stated reason changes, and one shuttleengine test folds into a new one. No signature changes.

**Commit:** 4: Shuttle's startup probe runs every poll interval with backoff and spaces gate dismissals
