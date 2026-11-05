# Card 24 — drift-retire-old-seats

**Delete:**
- `internal/driftengine#BaselineInputs`
- `internal/driftengine#BaselineTable`
- `internal/driftengine#captureReading`
- `internal/driftengine#VerifyInputs`
- `internal/driftengine#VerifyTable`
- `internal/driftengine#VerifyTold`
- `internal/driftengine#baselineStencilName`
- `internal/driftengine#verifyStencilName`
- `internal/driftengine#baselineTimeout`
- `internal/driftengine#verifyTimeout`
- `internal/driftengine#baselineSkills`
- `internal/driftengine#verifySkills`
- `internal/driftengine#autonomousVerifyMode`
- `internal/driftengine#operatorVerifyMode`
- `internal/driftengine#autonomousVerifyJudging`
- `internal/driftengine#operatorVerifyJudging`
- `internal/driftengine#acceptanceKeepsOutput`
- `contracts/stencils#DriftTemplateBaseline`
- `contracts/stencils#DriftTemplateVerify`
- `contracts/stencils/drift/drift-template-baseline.md`
- `contracts/stencils/drift/drift-template-verify.md`
- `internal/loomcli_test#TestDriftDeps_AcceptedSeamsKeepAcceptedTestsInTheRun`

**Edit:**
- `internal/driftengine#Deps`
- `internal/driftengine#Paths`
- `internal/driftengine#NewPaths`
- `internal/driftengine#CompareRequest`
- `internal/driftengine#RunCompare`
- `internal/driftengine#TestSeatTables`
- `internal/driftengine#TestRunCompare`
- `internal/driftengine/doc.go`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/loomcli#driftDeps`
- `internal/loomcli#driftTables`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/doc.go`
- `internal/testkit/envkit#driftDeps`
- `internal/battencli#intentionallyNil`
- `pattern/PATTERN-parent-directive.md`
- `internal/shedrecipe#driftTestEnv`
- `contracts/stencils#carriesMarker`
- `contracts/stencils#openingStencilViolations`
- `contracts/stencils#parentMarker`
- `contracts/stencils#editMarker`
- `contracts/stencils#editRuleSentences`
- `contracts/stencils#unboundedAskSentences`
- `contracts/stencils#containsAny`
- `contracts/stencils#registeredBodies`
- `contracts/stencils#TestStencils_OpeningStencilsRenderParentDirective`
- `contracts/stencils#TestStencils_OpeningStencilsRenderEditDirective`
- `contracts/stencils#TestStencils_NoEditRuleRestatement`
- `contracts/stencils#TestStencils_NoUnboundedAskOperator`
- `contracts/stencils#TestParentDirectiveScans_SyntheticRows`
- `internal/driftengine#TestLoadConfig`
- `internal/driftengine#baselineFixture.seedScopeRecord`
- `internal/driftengine#TestDriftBaseline`
- `internal/driftengine#TestDriftAwait`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine#newVerifyFixture`
- `internal/loomcli#TestWire_DarnRecipe`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Remove the Drift-Baseline and Drift-Verify agents' seats, stencils and seams, which no row reads since cards 16 and 18 (D14).

- The two table builders, their inputs, `VerifyTold`, `captureReading`, the stencil-name, timeout, skill and judging-text constants go; `baselineRole` and `verifyRole` stay as the removed roles `EndRoles` ends on entry (D4).
  The Drift-Verify seat's language skills and PATTERN implementer directive go with `VerifyTable`.
- `Deps` drops `BaselineTable`, `VerifyTable` and `CompareAccepted`; `loomcli`'s `driftDeps` drops their wiring and `driftTables` its `languageSkills`, which `wire` no longer passes; `envkit` and `battencli`'s `intentionallyNil` drop them.
- `Paths` drops `BaselineReport` and `VerifyReport`, so no code names `baseline.md` or `verify-report.md`.
  `TestLoadConfig`'s subtest "the durable paths lie under DirRel" drops both from its durable map; `baselineFixture.seedScopeRecord` drops its `report` parameter and its write of `baseline.md`, and its callers in `TestDriftBaseline`, `TestDriftAwait`, `TestDriftVerify` and `newVerifyFixture` drop the argument wherever card 16's rewrite kept the call.
- `TestWire_DarnRecipe`'s sub-run that reads `c.env.Drift.BaselineTable("")` reads `c.env.Drift.ExpectTable("")` instead, still asserting that `subject_reading` holds the decision record path and no `lym board get`.
- `CompareRequest` drops `ProfileDir`, which only the accepted profile used; `RunCompare` renders `{baseline_dir}` as the baseline directory always, and `TestRunCompare` drops its profile-directory row.
- The stencils `drift-template-baseline` and `drift-template-verify`, their embed vars, `entries` rows, `roleOpeningStencils` rows and marker rows go.
- `PATTERN-parent-directive.md`'s example of the Drift-Verify seat names the judge seat instead, and so does `parentdirective_test.go`'s file header comment, which names the Drift-Verify seat under `verify_interactive`: it names the judge seat under `judge_interactive`. The header is no member's, so the card lists every member of the file, as `edit-test-file`'s way forward asks for a header-only change; no member's body changes.
- `shedrecipe`'s `driftTestEnv` stops filling `BaselineTable`, `VerifyTable` and `CompareAccepted`.

`TestSeatTables` keeps the expect and judge rows only; `TestDriftDeps_AcceptedSeamsKeepAcceptedTestsInTheRun` goes with the seam it covered.

**ImpactSummary:** Deletes the two old agents' seats, stencils and three seams no row reads; loomcli, envkit and battencli's nil list drop the seams; the stencil registry loses two rows.
