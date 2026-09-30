# Fixer report — plan round 1

All six findings are fixed in the plan cards.
`lyx loom validate-plan` still reports `ok: true` after the edits.
Its one informational finding, that Card 2's Create target introduces a new unit, predates this round.

## Fixed

- **F1 (MEDIUM), the combined first criterion:** Card 3 now also edits `internal/loomcli/smoke_driverstrand_test.go`.
  `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` asserts zero `loom-status` strands after each of its three bootstraps, beside the driver-strand count.
  It is the one llm-seeded start whose driver strand really spawns, so the test now pins "adds none while the driver strand still spawns" in one session.
- **F2 (MEDIUM), failed reed `Up`:** Card 2 adds `TestSmokeStep_FailedReedUpRefusesWithBootstrapKind`.
  Card 3 adds `TestSmokeStart_FailedReedUpRefusesOnEitherDriver`, a go/llm table asserting exit 1, `ok: false`, and no watchdog or driver process.
  Both use a reed config with an invalid `mouse` value, which `Up` rejects before any tmux round trip, and Card 3 factors that fixture into one file-local helper.
  Card 2's ImpactSummary now mentions the preserved `bootstrap` refusal.
- **F3 (MEDIUM), the overview's second `start` description:** Card 3's docs/overview.md paragraph now also rewrites the "the bootstrap" bullet's status-strand clause by driver.
- **F4 (LOW), unpinned warning:** Card 1's two failure cases now capture the logger through `logger.SetOutput` and assert the warning, naming the guid in the remove case.
- **F5 (LOW), stale test-file prose:** Card 3 now edits `internal/loomcli/integration_driverbootstrap_test.go`'s header so the watcher-pane hazard is the go arm's.
  It also names `smoke_starttail_test.go`'s "only non-driver strand" clause explicitly.
- **F6 (NIT), tmux skip:** Cards 2 and 3 tell every new live-tmux smoke test to call `tmuxBinaryPath(t)` first.
  The failed-`Up` tests need no tmux and are exempt.

## Deferred

None.
