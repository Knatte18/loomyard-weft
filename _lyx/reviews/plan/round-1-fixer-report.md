# Round 1 fixer report — plan

Every finding fixed as proposed; `lyx loom validate-plan` returns `ok: true` with informational findings only.

## Changes

- **F1** (fixed as proposed):
  - `07-runner-engine-set.md`: the Intent says the `toldErr` join is a fail-loud wiring guard with no per-row test, since `shuttleengines.New` refuses an unregistered `cfg.Engine` (card 5's `build`) and card 1's `TestEngineSet_Lookup` pins the lookup's error.
    The `newParentReviewConfig` lifetime-error fallback is marked as the same guard.
    The unregistered-default rows are dropped from `TestNewRunner_RefusesUnusableToldPaths`, `TestNewDetachedRunner_RefusesUnusableToldPaths` and `TestNewParentReviewConfig_KeepAliveInterval`, which stay targets for the `shuttlefake.EngineSet` wrapping.
  - `11-fork-audit-capability.md`: `AuditEngineFrom`'s lookup-error return is stated as that guard, and the unregistered-default `TestAuditEngineFrom` row is dropped.
  - `12-orch-resume-context.md`: resume-context's unresolved default is stated as that guard, and the `NewEngineSet("missing", ...)` row is dropped.
  - No target list changed.
- **F2** (fixed as proposed): `plan:internal/modelspec#EngineNames` moved from card 4's Create list to card 5's, after `plan:internal/shuttleengines#Names`.
  Card 4's `EngineNames` paragraph became one sentence in card 5's Intent, and card 4's Intent opening keeps only the D11 half.
  The Card Index lines for cards 4 and 5 in `00-overview.md` are updated to the wording the fix gives.
- **F3** (fixed as proposed): card 14's Edit list names `internal/shuttleengine/seam_enforcement_test.go` in place of its `TestProviderSeamImportRule` member.
  A new Intent sentence says the file header states the module-wide rule: only `internal/shuttleengines` and an engine's own directory import a concrete engine.

## Disputed

None.

## Deferred

None.
