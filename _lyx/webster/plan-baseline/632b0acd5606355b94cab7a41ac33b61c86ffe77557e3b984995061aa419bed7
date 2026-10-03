# Card 19 — plankit-loom

**Edit:**
- `internal/loomcli/validate_test.go`
- `internal/loomcli/parity_test.go`
- `internal/loomcli/landingdeps_verify_test.go`
- `internal/loomcli/wiring_rework_integration_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomshed/fixture_test.go`
- `internal/loomshed/gates_test.go`
- `internal/loomshed/gatefindings_test.go`
- `internal/loomshed/prrework_test.go`
- `internal/planglyph/handle_test.go`
- `internal/planglyph/repo_test.go`
- `internal/shedrecipe/entries_prrework_test.go`

**Uses:**
- `internal/testkit/plankit/plankit.go`

**Intent:** Convert the loom-side hand-written plan writers and glyph-repo fixtures to `plankit` (D10): `loomcli`'s `planFixture`, `glyphPlanFixture`, `glyphRepoPlanFixture`, `planFixtureInvalidFormat`, `writeVerifyPlan`, `writeReworkPlan`, `writeGlyphRepoForCLITest` and `writeGlyphRepoForParityTest`;
`loomrecipe`'s `seedPlanFixture`/`planFixtureOverview`;
`loomshed`'s `seedPlanFormatFixture`, `seedGlyphPlanFixture`, `seedFormatInvalidPlanFixture`, `writeGlyphRepoFixture`, `writeOverviewOnlyPlanDir` and `seedReworkGlyphPlan`;
`planglyph`'s `writePlanFixture` and `writeFixtureRepo`;
and the plan written in `shedrecipe`'s `entries_prrework_test.go`.
Each converted fixture still produces an equivalent valid plan or file tree, so every test keeps its verdict.
Rejection fixtures (an unrecognized format, an overview with no cards, a bad `first_card:`) corrupt `plankit`'s rendered bytes or keep raw bytes that never spell the current format version.
After this card `format: 5` appears in no test file outside `internal/planparser` and `internal/testkit/plankit`;
a repo-wide grep for it in `_test.go` files is part of this card's own check.

**ImpactSummary:** Test fixtures only, in `loomcli`, `loomrecipe`, `loomshed`, `planglyph` and `shedrecipe`; every converted test keeps its assertions.
