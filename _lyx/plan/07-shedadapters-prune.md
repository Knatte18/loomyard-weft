# Card 7 — shedadapters-prune

**Edit:**
- `internal/shedadapters#`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/shedadapters`.

Every test file of the package is untagged, so there is no tier move; before D1 every non-candidate test sat in the "no coverage" list as `unclassifiable call`, and card 6's rerun brings all of them into judgment.
Fold first: the `TestReadRubric_*` family, the `TestParse*ProseAndUnknownKey` family, the `TestReadCirclingDecision_*` and `TestReadRoundFocus_*` pairs and the `TestSingleLLMProducer_*` outcome tests are likely tables, judged by reading.
Then delete or keep the remaining candidates, rerunning `-pkg ./internal/shedadapters` until no candidate outside "Kept" remains and no test is "failed".

The package has no `TestMain`, spawns nothing in its tests and keeps that shape.

**ImpactSummary:** Test files of one package and its report section only; production code is not expected to change.
