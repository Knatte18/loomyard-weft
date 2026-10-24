# Card 13 — testtiming-tiers

**Edit:**
- `cmd/testtiming#resolveTags`
- `cmd/testtiming#run`
- `cmd/testtiming#fileTier`
- `cmd/testtiming#TestResolveTags`
- `cmd/testtiming#TestScanSpawns`
- `cmd/testtiming#defaultRedundancyTags`
- `cmd/testtiming#TestRedundancyTags`
- `docs/benchmarks/running-tests.md`

**Intent:**
`cmd/testtiming` measures the disjoint tiers (D5).

Its untagged default mode runs `-tags tier0`, so it measures the `tier0` tests once they are tagged; `-full` maps to `tier1`, and the refusal of `-full` beside `-tags` names `tier1`.
`fileTier`, the spawn scan's tag classification, reads `tier1`, `tier2` and `tier3` beside `integration`, `tmux` and `llm`: a file naming `tier2` is tmux-tier, and a file naming only `tier0` counts as untagged.
The `-redundancy` mode's default, `defaultRedundancyTags`, becomes `tier0,tier1,tier2,integration,tmux`, every tier but `llm` under both spellings, so it still compiles every tier's tests but `llm`'s after the retag.
`TestResolveTags` asserts the new mapping, `TestRedundancyTags` the new default, and `TestScanSpawns` gains rows for a `tier2` file and a `tier0` file.
The tool's usage comment and `docs/benchmarks/running-tests.md` name the tier tags and say `tier1` no longer runs `tier0`'s tests.

**ImpactSummary:** Changes the timing tool's default and `-full` tags and its spawn scan's classification; no other caller.
