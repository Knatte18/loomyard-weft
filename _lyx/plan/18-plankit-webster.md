# Card 18 — plankit-webster

**Edit:**
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/verbs_test.go`

**Uses:**
- `internal/testkit/plankit/plankit.go`

**Intent:** Convert the webster-side hand-written plan writers to `plankit` (D10): `websterengine`'s `seedPlanDir`, `seedRewritingPlanDir`, `seedRunPlanDir`, `writeRecordPlanDir` and `seedDriftPlanDir`, `webstercli`'s `seedValidPlanDir`, `seedMissingIntentPlanDir`, `seedGlyphPlanDir`, `seedTwoCardGlyphPlanDir` and `seedTwoCardPlan`, plus any other valid-plan writer in these files, such as the inline overview in `rebaseline_test.go`.
Each converted fixture still produces an equivalent valid plan: same cards, targets, approval and plan-level sections, so every test keeps its verdict.
A fixture that tests rejection (`seedMissingIntentPlanDir` and the like) takes `plankit`'s rendered bytes and corrupts the one part under test, or keeps raw bytes that never spell the format version.
`cliwire`'s `seedPlanDir` and `webstercli`'s `seedStandalonePlanDir` stay as they are: they write the bytes `placeholder` only to satisfy `planDirHasContent`, spell no plan format, and so are not plan writers D10 reaches.
After this card no file listed here contains `format: 5`.

**ImpactSummary:** Test fixtures only, in `websterengine` and `webstercli`; every converted test keeps its assertions.
