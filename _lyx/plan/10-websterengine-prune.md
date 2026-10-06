# Card 10 — websterengine-prune

**Edit:**
- `internal/websterengine#`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/websterengine`, starting with a rerun of `-pkg ./internal/websterengine`, since card 9 moved and merged tests.

Fold first: the `TestArchiveRunRecord_*`, `TestClassify_*`, `TestDistill_*` and `TestFingerprint_*` families in the committed report are each one behavior over several inputs and are likely tables, judged by reading.
Then delete or keep the remaining candidates, rerunning until no candidate outside "Kept" remains and no test is "failed".

**ImpactSummary:** Test files of one package and its report section; production code is not expected to change.
