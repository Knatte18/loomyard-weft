# Card 4 — The startup probe backs off from one poll interval, and gate dismissals keep their gap

**Create:**
- `internal/shuttleengine#dismissalGap`
- `internal/shuttleengine#playInputsSent`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#playInputs`
- `internal/shuttleengine#startupTickCap`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/fixture_test.go`

**Intent:**
Decisions D5 and D6, with D5 as superseded by the parent's backoff decision: every shuttle start, and so every batten window, stops taking about 11 seconds because `awaitStartup` sleeps a whole 10-second probe interval, while a startup gate still never gets a second key from a capture taken before Claude redraws.
The two land in one card because a faster probe without the dismissal gap would replay a gate's keys every second.

`dismissalGap(cfg)` returns `pollInterval(cfg)` times `cfg.LivenessEveryNPolls`, the multiple floored to 1 exactly as `Run.Wait` floors it: 10 s under the shipped template.

`awaitStartup` (`PATTERN-daemon-wakeups` backoff rule; the background is not amended):
- The base is `pollInterval(cfg)` and the ceiling is `dismissalGap(cfg)`.
- The first sleep is the base.
  After each later probe, the sleep is the base when the capture `checkLivenessTick` recorded in `run.lastStartupCapture` differs from the one the previous probe left there, and otherwise double the previous sleep, capped at the ceiling.
  A probe whose capture failed, or that hit a reed-status error, leaves the capture unchanged and so backs off.
- `maxTicks` is `startupTickCap(window, pollInterval(cfg))`: the base interval yields the most ticks, so the count cap still bounds the loop under a clock that never advances.
- Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture; no exit, return or `allOutputFilesExist` call changes, so `completionsignal_enforcement_test.go`'s counts hold.
- No sub-second literal is added.
- The "Probe cadence" paragraph of its doc comment is rewritten, one sentence per line.
  It states the backoff, base and ceiling.
  It says a ready provider shows within the first one or two probes because Claude's boot redraws the pane and resets the backoff.
  It says gate keys are spaced by `checkLivenessTick`'s dismissal gap rather than by the probe cadence.

`startupTickCap`'s doc comment says its interval is the startup probe's base interval.

`Run` gains `dismissedAt time.Time` and `dismissed bool`: when this run last played a startup-gate dismissal that sent a key, on the run's clock.

`playInputsSent` is `playInputs`'s body, returning as well how many inputs were delivered before it returned, so a play whose first key failed reports 0.
`playInputs` becomes `_, err := playInputsSent(...)`, so its callers are untouched.

`checkLivenessTick`'s `StartupTrustPrompt` branch plays only when `!run.dismissed` or `run.clock.Now().Sub(run.dismissedAt) >= dismissalGap(cfg)`:
- When it plays, it asks `TrustDismissSequence` for the inputs and plays them through `playInputsSent`.
- Any play that sent at least one input, failed partway or not, sets `dismissed` and `dismissedAt` to the time the play began.
  An empty sequence or a play whose first input failed sets neither, so the next probe may dismiss.
- A probe inside the gap that still sees a gate plays nothing, does not call `TrustDismissSequence`, and logs at Debug that the dismissal is held for the gap.
- Every branch still falls through to `classifyStartupWindow`.
Bound, in its doc comment: the throttle only removes key presses and never adds one, and the startup deadline and the tick cap still end a gate that never clears.
`Run.Wait`'s own startup probe already runs at least one gap apart, so under the at-or-past rule the throttle never holds a dismissal there.

`template.yaml`'s `liveness_every_n_polls` comment adds that the same multiple of the poll interval also spaces startup-gate dismissals and caps the startup probe's backoff; no key is added.
`doc.go`'s closing startup paragraph gains the same two facts: the startup probe starts at the poll interval and backs off to the dismissal gap while the pane does not change, and gate dismissals are at least one dismissal gap apart.

Tests, in `startup_test.go`, all on the injected fake clock with `PollIntervalMS: 1000`, `LivenessEveryNPolls: 10`:
- `TestStartup_ReadyProbeTiming` replaces its "probe cadence matches Wait" row with two rows; its doc comment and the file's header comment drop the matches-Wait claim.
  Its `//testtiming:keep` reason is reworded to what the rows pin: no sleep when ready on the first probe, the base interval while the pane redraws, and the backoff to the dismissal gap while it does not.
  - "a pane that redraws between probes is probed every poll interval": distinct captures on each probe, script pending, pending, ready; 3 status calls, 2 s elapsed.
  - "an unchanged pane backs off to the dismissal gap": one constant non-empty capture, six pending probes then ready; sleeps 1, 2, 4, 8, 10 and 10 s, so 7 status calls and 35 s elapsed.
- `TestStartup_TickCap` computes `wantTicks` from `pollInterval(cfg)`.
- `TestStartup_UndismissableGateUntilWindowExpires` becomes the table-driven gate-never-clears test, renamed `TestStartup_GateNeverClearsUntilWindowExpires`; each row builds its own reed, engine and config.
  Each row asserts the start fails wrapping `ErrNotStarted` with the record's outcome died and one strand removal, and the virtual times of the plays.
  Its doc comment and `//testtiming:keep` reason say it pins a gate that never clears: its dismissals' virtual times, at most one play per dismissal gap, until the window expires.
  - "a gate whose accepting option cannot be located sends no key": today's case unchanged, the empty-sequence branch, with no `SendKey` call and no dismissal log line.
  The other three rows use `StartupTimeoutS: 30`, the startup script always `StartupTrustPrompt` and a distinct capture on every probe so the backoff stays at the base.
  - "a gate that never clears is dismissed once per gap": one `Enter` at each of 0, 10, 20 and 30 s, so 4 `SendKey` calls over the 32 probes.
  - "a play whose first key fails leaves the next probe free": `SendKeyErr` set, so a `SendKey` call on every probe.
  - "a play that sends its first key and fails a later one starts the gap": a test engine whose dismiss sequence is two keys and a reed that fails the second, so plays at 0, 10, 20 and 30 s only.
  The package gains no top-level test.

**ImpactSummary:** Every shuttle start's pre-ready probe now runs from one second and backs off to the dismissal gap, and startup-gate keys, in `awaitStartup` and in `Wait`'s startup probe, are at least one dismissal gap apart; `playInputs`' callers see no change.

**Commit:** 4: The startup probe backs off from one poll interval, and gate dismissals keep their gap
