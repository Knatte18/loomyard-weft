# Card 12 — reedengine-prune

**Edit:**
- `internal/reedengine#`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/reedengine`, starting with a rerun of `-pkg ./internal/reedengine`, since card 11 moved and merged tests.

Before D1, untagged reed tests that reach a fake through a method the scan matched by name were excluded as spawning; card 6's rerun judges them.
Fold first: the `TestPlanLayout_*`, `TestStageLaunchScript_*`, `TestCleanClaudeEnv*` and `TestPaneGeneration*`/`TestParsePaneGeneration` families in the committed report are likely tables, judged by reading.
Then delete or keep the remaining candidates, rerunning until no candidate outside "Kept" remains and no test is "failed".
Tests of `tmux` files stay outside judgment (D1); they are still read for folding (D5).

Do not touch `internal/reedengine/render`: it is a separate package outside this task.

**ImpactSummary:** Test files of one package and its report section; production code is not expected to change.

**Verify:**
go test -tags tmux -count=1 ./internal/reedengine/
