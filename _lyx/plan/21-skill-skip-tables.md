# Card 21 — One table per skill-skip surface

**Edit:**
- `internal/shuttleengine/startup_test.go`
- `internal/orchengine/watcher_test.go`

**Intent:**
Finding F2, applied to the other new tests of the same kind: the two causes for skipping a skill are many items of one kind, tested one each on two surfaces.

- `TestStartup_SkillWithoutTurnEndSkippedAtTimeout` and `TestStartup_UnknownSkillSkippedAtOnce` become one table-driven startup test with a row per cause.
  Each row keeps its exact typed sequence and its elapsed-time assertion: at least the timeout for the silent skill, zero for the unknown one.
- `TestWatcher_UnknownSkillSkippedAndLogged` and `TestWatcher_SkillLoadTimeoutSkips` become one table-driven watcher test with a row per cause.
  Each row keeps its assertions: `ReloadStep` advancing past the skipped skill, no skip before the timeout for the silent skill, and the calls after the clear.

**ImpactSummary:** test-only: two pairs of skip-cause tests each folded into one table test; coverage unchanged.

**Commit:** 21: shuttle, orch: one table per skill-skip surface
