# Card 8 — shuttleengine-prune

**Edit:**
- `internal/shuttleengine#`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/shuttleengine`.

Every test file is untagged, so there is no tier move; before D1 most tests were `unclassifiable call`, and card 6's rerun judges them.
Fold first: the `TestSpec_Validate_*` family (defaults, pass-through and untouched fields of one `Validate` call), the `TestRunDirRoot_*` pair, the `TestFindRunByStrand_*` hit/miss pair and the `TestLoadConfig_*` tests are likely tables, judged by reading.
Then delete or keep the remaining candidates, rerunning `-pkg ./internal/shuttleengine` until no candidate outside "Kept" remains and no test is "failed".

`internal/testkit/shuttlefake` is the package's existing fake; extend it only where a test of another package also needs the extension, per `PATTERN-testkit`.
Do not touch `internal/shuttleengine/claudeengine`: it is a separate package outside this task.

**ImpactSummary:** Test files of one package and its report section only; production code is not expected to change.
