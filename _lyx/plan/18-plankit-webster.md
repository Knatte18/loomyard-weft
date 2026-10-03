# Card 18 — plankit-webster

**Edit:**
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/template_test.go`
- `internal/webstercli/cli_test.go`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/wiring_test.go`
- `internal/cliwire/cliwire_test.go`

**Uses:**
- `internal/testkit/plankit/plankit.go`

**Intent:** Convert the webster-side hand-written plan writers to `plankit` (D10): `websterengine`'s `seedPlanDir`, `seedRewritingPlanDir`, `seedRunPlanDir`, `writeRecordPlanDir` and `seedDriftPlanDir`, `webstercli`'s `seedValidPlanDir`, `seedMissingIntentPlanDir`, `seedGlyphPlanDir`, `seedTwoCardGlyphPlanDir`, `seedTwoCardPlan` and `seedStandalonePlanDir`, and `cliwire`'s `seedPlanDir`, plus any other valid-plan writer in these files.
Each converted fixture still produces an equivalent valid plan: same cards, targets, approval and plan-level sections, so every test keeps its verdict.
A fixture that tests rejection (`seedMissingIntentPlanDir` and the like) takes `plankit`'s rendered bytes and corrupts the one part under test, or keeps raw bytes that never spell the format version.
After this card no file listed here contains `format: 5`.

**ImpactSummary:** Test fixtures only, in `websterengine`, `webstercli` and `cliwire`; every converted test keeps its assertions.
