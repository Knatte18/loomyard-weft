# Card 4 — describe-function

**Create:**
- `plan:internal/landingshed#WriteBeamDescription` -> `func WriteBeamDescription(deps BeamDescribeDeps) error`

**Edit:**
- `internal/landingshed#beamDescribeProducer.Call`
- `internal/landingshed#BeamDescribeDeps`
- `internal/landingshed#TestBeamDescribe`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/landingshed#beamDescription`

**Intent:** Turn the beam description writer into a function a verb calls (D5): it stays in `landingshed`, and the Beam-Describe producer, removed with its row by card 5, calls it until then.

- `WriteBeamDescription` does what `beamDescribeProducer.Call` does today, with no context: it refuses a deps with an unset field, naming it; reads the entry's title and brief, the project record's added decisions and the beam's first-parent commits; writes `summary.md` whole at `DescriptionPath`; validates it with `summaryparser.ValidateDescription`; and commits it through `Commit`. A read, write, validation or commit failure is an error naming the step.
- A description that fails validation is an error naming each finding and the way forward `fix the title or brief with "lym board upsert", then run "lym beam describe <slug>" again` (D5), since the beam's run no longer claims its entry while it waits; the old text naming `lym batten cancel` and `lym beam land` goes.
- `beamDescribeProducer.Call` keeps its context checks and returns `WriteBeamDescription`'s error wrapped as today.
- `BeamDescribeDeps`' doc names the verb as its caller; its `Commit` doc says a repeat commit over unchanged content is a no-op, as now.

Tests (tier 0): `TestBeamDescribe` drives `WriteBeamDescription` directly for its existing rows (each commit kind, the decisions section, the title cut, a re-run rewriting the same bytes, each failing seam) and asserts the validation refusal names `lym board upsert` and `lym beam describe <slug>` and no longer `lym batten cancel`; one row keeps the producer path, asserting `Call` returns the function's error.

**ImpactSummary:** one new exported function in landingshed and a reworded validation refusal; the producer's behaviour is otherwise unchanged, and only `TestBeamDescribe` asserts the old text.

**Commit:** 4: Make the beam description writer a function
