```yaml
lens: row-validity
outcome: reported
```

Scope: the row-validity lens over the seven cards, checking each planned test row and new test.

**BLOCKING**
- **Card 6, `TestGateHolds`: the failing-`LoadConfig` row cannot run through the harness.**
  `d.startReady` calls `d.deps.LoadConfig()` on every scheduling tick and returns `load beam.yaml: …` on failure (`internal/dobbyengine/starts.go:93-96`).
  `passHarness.tick` then calls `t.Fatalf` on any tick error (`internal/dobbyengine#passHarness.tick`).
  So the row "a failing `LoadConfig` … posts nothing" aborts the test instead of asserting.
  Fix: in that row, call `d.gateHolds(ctx, now)` directly. Alternatively, express the failure through the `Holders` seam, which is where D6's "failed gate config load" lives in card 7's wiring, and drop the beam-config failure row.

**MEDIUM**
- **Card 3, `TestRunWatch` retry row: it cannot tell "no sooner than a probe interval" from "every tick".**
  The harness's `tick` advances exactly 10s (`internal/dobbyengine#passHarness.tick`) and `probe` is 10s (`internal/dobbyengine#newPassHarness`).
  `NextTry = now + d.probe` therefore equals the next tick's `now`, and since "a line whose `NextTry` is after now waits", the line is retried on every tick either way.
  Fix: tick the retry row with `internal/dobbyengine#passHarness.tickAfter` at a gap below the probe interval, and assert no second attempt until a probe interval has passed.
- **Card 3, `TestClearDoneRuns`: the "cleared run's `Watches` entry is gone" assertion passes vacuously.**
  The table's setup seeds only `LandNotices` and `CountedLandings` (`internal/dobbyengine/donerun_test.go` state setup in `TestClearDoneRuns`).
  A cleared row starts with no `Watches` entry, so "gone" holds even if the clear never drops it.
  The card must add a table field that seeds a `Watches` entry owing nothing on the cleared rows, alongside the owing-a-line field of the new row.
- **Card 3, `TestRunWatch` rows that depend on re-reading the child do not state they move `ChildStatusModified`.**
  This covers the episode row ("blocked again at the same row with a longer history"), blocked→failed, the pause lift and the teardown vanish.
  The card reads the child only when `RunView.StatusModified` or `ChildStatusModified` differs from `StatusSeen` and `ChildSeen`.
  With hand-built `RunView`s that leave those zero, the child is read once and every later child reading is ignored.
  The episode row then posts no second line, and the absence would look like a code bug.
  The card should state that each tick's `RunView` carries a new `ChildStatusModified` whenever its child reading changes.
- **Card 7: the `dobbyGateHolders` branches go untested.**
  These are the `gateslot.LoadConfig` failure, the `Pool.Holders` failure, and the tier→`TierTimeout` mapping, including an empty or undeclared tier.
  Card 7's tests touch only `gateHoldTags` and `Post`.
  D6's "a failed gate config load … posts nothing and warns" and the per-tier bound are pinned only through card 6's fake, so a wrong `cfg.Tiers[...]` lookup or unit (minutes) would ship green.
  A row in an existing hubforge integration test, as `newDetachRig` sets one up, could hold a tier slot and assert `deps.GateHolds.Holders()` returns its `TierTimeout`.
- **Card 7, `TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy`: the no-supersede assertion may be confounded by the existing rows.**
  The test's two rows (prime dobby and beam dobby) already post two run notices for `task1` under one topic, `dobby-run-notice/task1` (`internal/beamcli#runNoticeTags`; `internal/beamcli/dobbynoticestamp_integration_test.go`, the `tests` loop).
  If creel supersedes across those senders, the new "no `undelivered` with `creelengine.SupersededReason`" assertion (`internal/creelengine/verbs.go:301`) fails on existing behaviour, not on the gate-hold or watch lines.
  The card should scope the assertion to the gate-hold and watch subjects, or state why the two run notices do not supersede.

**LOW**
- **Card 5, `TestGateTest_TierHolderRecordsItsRunMarker`: the new tier-1 top-level test repeats existing setup.**
  `TestGateTest_DetachedRun`'s first subtest already holds a tier run going with `newRelease` and `releaseEnv` before it releases (`internal/gatecli/detachfront_integration_test.go:374-382`).
  Reading `rig.pool.Holders()` there before `release()` would pin the marker with no second hub.
  Separately, the card's `Uses` omits `newRelease` and `releaseEnv`, which its stated "waits on a release file" needs.
- **Card 3, the not-posted row: no tick count is stated.**
  To fail if attempts were wrongly counted, the row needs more than `runNoticeMaxAttempts` (3) not-posted ticks before the post succeeds. The card does not say so.
- **Card 6, the marked-process row: the fake's shape is unstated.**
  The fake `Tree` must answer per PID, both the holder's tree and each marked PID's own entry, since the card samples each marked PID through `Tree(pid)`. The card does not say the fake is keyed by PID.
- **Card 2: every existing row's want changes, not one row.**
  The batten status constants in `TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold` (`running` and `keptCancel`) carry no `history` or `error`.
  Giving them a known history and reason changes every row's want. That is fine, but the card should say the shared constants gain them.

Tool calls:
1. `internal/dobbyengine/donerun_test.go` `TestClearDoneRuns` and the `LoadConfig` callers in `starts.go` and `tidy.go`. My context had only the test's name, and I needed which steps fail the tick on a config error.
2. The gating of `starts.go` and `tidy.go`, the detach rig grep and the beamcli test locations. I needed whether `startReady` always loads config, and where the cards' named tests live.
3. The `detachfront_integration_test.go` rig helpers and `TestGateTest_DetachedRun`. My context lacked the rig's slot-holding mechanism.
4. The release helpers and `TestGateTest_DetachedRun`'s first subtest. I needed whether an existing test already holds a tier run.
5. `dobbyruns_integration_test.go`, `TestRunNoticeTags` and `TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer`. My context lacked these tests' bodies.
6. `creelengine`'s `SupersededReason` and the body of `TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy`. I needed whether the supersede assertion is sound against the existing rows.
