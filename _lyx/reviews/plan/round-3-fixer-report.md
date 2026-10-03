# Fixer report — plan, round 3

## Fixed

- **F1:** card 3 lists `internal/loomcli/wiring_test.go` in its Edit list.
  Its Intent now names `reflectFrictionRow`'s condition as the field's only production reader and `TestArmAt_RecordsTheArmingVerb` as its only other reader, and removes that test with the field.
  The ImpactSummary names the removed test.
- **F2:** card 6 lists `internal/loomcli/approve_awaiting_test.go` in its Edit list.
  Its Intent removes both tests that call `shouldReflectFriction` and moves the awaiting property to `halt_test.go`, whose `loomPostRun` bullet now also asserts that `RunAwaiting` writes no note, spawns nothing and reports `skipped`.
  The ImpactSummary names `loomPostRun` as the production caller and both test files.
- **F3:** the overview's untagged `go test` line in `## verify:` adds `./cmd/lyx/...`, so `TestLyDriveSkill_IsRecipeBlind` runs against card 7's rewrite.
- **F4:** card 3 renames `TestReflectFrictionRow_ReflectsWhenArmedForRun` to `TestReflectFrictionRow_ReflectsWhenTierTwoOn`, its doc naming Tier 2 on instead of an armed verb.

A repo-wide grep of every other symbol, field and test the cards remove or rename (`archiveFrictionDir`, `deleteStaleReport`, `TestStepEnvelope_KeySetIsExactlySixteen`, `StepEnvelope`'s and `stepErrFields`' callers, `TestReflectFrictionRow_SkipsWhenArmedForStep`, `newRelativeFrictionCLI`, the ly-drive row names) found no other reference outside its card's targets.

`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` findings only.
`approved:` stays `false`.

## Deferred

None.
