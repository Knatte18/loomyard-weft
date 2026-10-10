# Fixer report — plan, round 1

Every finding was validated against the tree and the decision record and fixed.
`lyx loom validate-plan` exits 0 on the fixed plan.

## Fixed

- **F1** (card 4): `internal/reedengine/watchdog_integration_test.go` joins `Edit:`.
  `Intent` says `fastWatchTiming` drops its `SignalTick` line and comment, the no-self-trigger test's `time.Sleep(10 * timing.SignalTick)` sleeps a fixed duration of its own, and its `startWatchLoop` calls pass `OpenFileWatch`.
  Confirmed at `watchdog_integration_test.go:33` and `:346`.
- **F2** (cards 10, 11): both list `cmd/lyx/testdata/test-budget.yaml`.
  Card 10 raises `internal/shell` by its one test (11 of 11 today), and card 11 raises `internal/orchengine` by its own tests on top of card 9's raise (126 of 126 today).
- **F3** (card 7): `internal/shuttleengine/claudeengine/doc.go` joins `Edit:`.
  `Intent` names the `SessionStart` row in the payload mapping, the hook in the recording-hook list, and the exception to "No recording hook prints to standard output" (`doc.go:84`).
- **F4** (card 8): `Intent` gains a `Bound:` sentence.
  A ready reading beside a needle or a missing box can reach an unreported screen state such as a menu, bounded by the landed check and by the gate typing only the watcher's own text (D7's Bound).
- **F5** (card 8): `readiness_test.go` is dropped from `Create:`, and `judgeReadiness` has no test of its own.
  `TestRunner_SessionIdle_ReturnsScriptedClassification` becomes the per-state-and-cause table over `Runner.SessionIdle`, keeping its existing pane-fallback rows.
  `TestSend_WaitsForIdleSession` repins its existing draft and unmatched-turn-start rows to the reading's reasons and gains only the needle row.
  Card 8 now adds no top-level test, so its now-unused `test-budget.yaml` target is removed.
- **F6** (card 5): `nameRepairTiming` carries an `After` timer seam, filled with `time.After` by default.
  The `watchNames` test records the requested waits through it and asserts that `repairNames`'s answer chooses the next wait, which the `NextWakeCadence` table cannot show, and reads no wall clock.
- **F7** (card 14): both figures come from a scratch hub cloned with `lyx fabric clone --into`, with three worktrees brought up by `lyx reed up`.
  The before figure comes from the production binary built from `main`, and the after figure from the `./deploy-dev` build, each started in the foreground against the scratch hub.
  The pid is found by the scratch hub's path, nothing of the operator's hub is touched, and the sessions end with `lyx reed down`.
- **F8** (cards 7, 13): card 7's `Intent` gains a `Bound:` sentence: `orchcli` is the only setter, and the command runs only at a compaction of the orch run.
  Card 13's `Intent` states that the annotation enforces announcement, not absence, bounded by the reason being on the line for review.
  It also says a bare `//lyx:one-shot` with no reason is refused, which D2's `//lyx:one-shot <reason>` form supports, and a crafted-snippet case covers it.
- **F9** (card 4): the floor assertion goes into `DefaultTimingMatchesTheSixConstants`, and no new test is added.
- **F10** (card 6): `internal/reedcli/doc.go` is created, holding the package comment and the discovery cadence, the signal wake with its base-cadence bound, and the longer idle-exit and reap delay.
  I first tried the reviewer's main fix, adding `internal/reedengine/doc.go` to card 6.
  The gate refused that as `delete-before-reference`, because that doc still names `watchdogSignalTick` (`doc.go:684`) until card 5's delete lands, so I used the reviewer's alternative, which the record's Scope-In docs line licenses for `reedcli`.
- **F11** (card 13): the detector cases are subtests of the one `//lyx:guard` test, and `cmd/lyx` is raised by one.
- **F12** (overview): the shared decision now covers `reedcli`'s production files only, and states that a `reedcli` test building the channel imports `fswatch` for its `Event` type.
- **F13** (card 5): the `DiscoverSignalFileName` pin is dropped, and `watchdog_test.go` keeps the `NextWakeCadence` table.

## Disputed

None.

## Deferred

None.
