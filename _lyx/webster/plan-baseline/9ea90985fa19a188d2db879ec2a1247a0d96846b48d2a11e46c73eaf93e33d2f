# Card 16 — fabricengine-prune

**Edit:**
- `internal/fabricengine#`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/fabricengine`, starting with a rerun of `-pkg ./internal/fabricengine`, since card 15 moved and merged tests.

This is the largest package of the six, and D1 brings most of its integration tests into judgment for the first time, so the hand read under D4 is the bulk of this card.
Fold first: the `TestCorrIndex_*`, `TestMergeErrors_*`, `TestMergeVocabulary_*` and `TestGate_*` families in the committed report are likely tables, judged by reading.
Then delete or keep the remaining candidates, rerunning until no candidate outside "Kept" remains and no test is "failed".
fabricengine tests that install a git hook or alias are the ones D4's "does git run module code" question is about; keep such a candidate with a reason naming the hook.

**ImpactSummary:** Test files of one package and its report section; production code is not expected to change.
