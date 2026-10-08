# Plan review round 3 — fixer report

All four findings validated against the code and fixed; part of F3's premise is false and is recorded under Disputed, with the rest of F3 fixed.
`lyx loom validate-plan` passes after the fixes (`ok: true`, informational `create-new-unit` notes only).

## Fixed

- **F1 (MEDIUM, scope):** confirmed: `smoke_lifecycle_test.go:300` asserts `pane_top` `0` for the sole Selvage pane, and `strandops_integration_test.go:79` and `attachgeometry_integration_test.go:250,369` assert the row-0 pane's height equals `cfg.CollapsedRows`.
  Card 5 now lists `internal/reedengine/strandops_integration_test.go` and `internal/reedcli/smoke_lifecycle_test.go`, and its Tests paragraph states that the row-0 height checks (`TestAttachGeometry_ExactLayoutAndRowBudgets`, `assertAttachGeometryRowBudgets`, the replacement-top-pane check) expect `cfg.CollapsedRows` minus the title row, and the sole-Selvage check expects `pane_top` `1`.
  Card 8 states that those `attachgeometry_integration_test.go` height checks keep that value, which its adjusted row-0 pin now holds after a resize too.
- **F2 (MEDIUM, scope):** confirmed: `attachgeometry_integration_test.go:202,296,396,488` assert a 10-element chain, and `:205,299` take the layout from `argv[len(argv)-1]`.
  Card 8 now states that the chain-shape checks find the chained form by its `select-layout` element and take the planned layout string from the argument after it, with the 5-element bare-form checks unchanged.
  Card 9 adds `internal/reedengine/attachgeometry_integration_test.go` to its Edit list and states that those checks still hold once its record entry leads the chain and its re-zoom entry ends it.
- **F3 (MEDIUM, consistency):** confirmed for `mergeresolve` (`TestBuildConflictSpec_SkillsAndParentDirective` asserts no `Role`), the recovery spec (no recovery test inspects the built spec) and the bouncer judge spec (`TestBouncer_PromptComposition` asserts `Prompt` and `Skills`, not `Role`).
  Card 12 now names `TestBuildConflictSpec_SkillsAndParentDirective` as the test that gains the `Segment` assertion beside `Skills`.
  Card 13 now names a test per builder: `TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand` via `shuttlefake.Reed.AddedSpecs` for the recovery spec, `TestBouncer_PromptComposition` for both the seed and judge specs, and `TestSpawnedRoles_SkillAndParentDirective` for both treadle judges and targeting.
  Both cards state that they add no top-level test function, so no test-budget row changes and neither card needs the budget file.
- **F4 (LOW, consistency):** card 3 drops `paneoptions_test.go` from its Create list.
  The extended `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` fixtures now cover the four name shapes, so `@strand` pins `strandLabel` through the real launch path.

## Disputed

- **F3, the treadle targeting part:** the finding says the targeting spec's role is asserted only in `treadleengine/engine_test.go:1078` and that `judge_test.go` "covers only the judge call", and asks to add `engine_test.go` to card 13.
  The premise is false: `internal/treadleengine/judge_test.go`'s `TestSpawnedRoles_SkillAndParentDirective` already calls `runTargeting(...)` in its `"targeting"` table entry beside the circling and milestone judges, and asserts on the captured `sh.spec`.
  Card 13 therefore extends that function for `targeting.go` and leaves `engine_test.go` out of its Edit list.

## Deferred

None.
