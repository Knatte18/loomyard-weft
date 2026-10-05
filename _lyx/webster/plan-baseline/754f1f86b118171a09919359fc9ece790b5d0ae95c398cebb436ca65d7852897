# Card 21 — One table per skill-skip surface

**Edit:**
- `internal/shuttleengine/startup_test.go`
- `internal/orchengine/watcher_test.go`

**Intent:**
Finding F2, applied to the other new tests of the same kind: the two causes for skipping a skill are many items of one kind, tested one each on two surfaces.

- `TestStartup_SkillWithoutTurnEndSkippedAtTimeout` and `TestStartup_UnknownSkillSkippedAtOnce` become one table-driven startup test with a row per cause.
  A row is `spec`, `endsTurn` and `unknown` (the lists passed to `skillStartFixture`), `wantTyped` (the exact typed sequence) and `elapsed`, a check on the virtual elapsed time of one of two kinds: `atLeastTimeout` or `zero`.
  The silent row's `spec` sets `SkillLoadTimeout` to 50ms and its `elapsed` is `atLeastTimeout`, asserting `elapsed >= spec.SkillLoadTimeout`; the unknown row's `spec` sets no timeout and its `elapsed` is `zero`, asserting `elapsed == 0`, so an unknown skill never waits out the timeout.
  The silent row's `wantTyped` is `LOAD:slow|LOAD:b|do the task` and the unknown row's is `LOAD:ghost|LOAD:b|do the task`, as the deleted tests asserted.
- `TestWatcher_UnknownSkillSkippedAndLogged` and `TestWatcher_SkillLoadTimeoutSkips` become one table-driven watcher test with a row per cause.
  A row has these fields:
  - `unknown`: the skills `skillUnknown` reports, set on `e.s.skillUnknown`.
  - `advances`: the clock advances applied in order, each followed by a tick; after each advance but the last the row asserts `ReloadStep` is still 0.
  - `endTurns`: the turn names ended after the skip.
  - `final`: the closing check, one of `reloaded` (`e.assertReload("clear", e.state().LastHandoff)`) or `skippedOnly` (the calls after the clear are two long with `skill:ly:board` second).
  Every row starts with `e.withSkills()`, `e.reachClearing()` and one tick that types the first skill, then applies its `advances`, ticking once more after each, and for a row with none ticks once more so the skip lands; it then asserts `ReloadStep` is 1.
  The unknown row sets `unknown` to `scribe:prose`, no `advances`, `endTurns` of `t2` and `resumed`, and `final` of `reloaded`.
  The timeout row sets `advances` of 99s and 2s, no `endTurns` and `final` of `skippedOnly`, as the deleted tests did.
  The table body is one short loop with one branch on `final`, and both causes fold into it.

**ImpactSummary:** test-only: two pairs of skip-cause tests each folded into one table test; coverage unchanged.

**Commit:** 21: shuttle, orch: one table per skill-skip surface
