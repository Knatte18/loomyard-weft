# Fixer report — plan, round 1

Every finding was checked against the code and holds; all five are fixed.

## Fixed

- **F1** (MEDIUM, scope) — Card 6's sandbox sites now name the `SANDBOX-CORE-SUITE.md` operating-model sentence (line 42), F11's verb list (bare `unwire` becomes `unwire --apply`, so the dirty-worktree matrix exercises the destructive path) and F15's alternation in place of F6, which never mentions unwire.
- **F2** (MEDIUM, scope) — Card 6 adds `internal/fabricengine/doc.go` under `**Edit:**`, and its Intent states that the `Unwire` paragraph names the `apply` parameter and says nothing is removed without it, with the result reporting what an applied run would remove.
- **F3** (MEDIUM, design) — Card 6 drops the `**Create:**` of `internal/fabriccli/unwire_integration_test.go` and its `cmd/lyx/testdata/test-budget.yaml` target and raise.
  It now edits `internal/fabriccli_test#TestRunCLI_HubMutationScenario` with one envelope-only step, placed before the hub-damaging steps: add a pair, run bare `unwire` there (exit 0, `"applied": false`, empty `mutations`), run `unwire --apply` (exit 0, `"applied": true`), then remove the pair.
  `00-overview.md`'s `test-budget-raises` decision now applies to cards 5 and 9 only.
- **F4** (MEDIUM, design) — Card 5 keeps the value-shape table in `floor_test.go` and trims the new `TestReconcileAll` rows to one below-floor row per module (`shuttle`, `orch`, `loom`), each checking the dry-run `Migrated` line with no write and the `--apply` write at the floor.
  The null, at-floor, env-marker and `abc` rows are dropped there.
- **F5** (LOW, scope) — Card 5 adds `internal/shuttleengine/doc.go` and `internal/orchengine/doc.go` under `**Edit:**`, each gaining one sentence on `MigrateConfig`; the Intent notes that `loomengine` has no `doc.go`, so its hook's doc comment carries it.

## Disputed

None.

## Deferred

None.
