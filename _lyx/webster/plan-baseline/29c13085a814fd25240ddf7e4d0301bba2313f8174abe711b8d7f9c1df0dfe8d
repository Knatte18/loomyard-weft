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
- `cmd/testtiming#prebuildLyx`
- `cmd/testtiming#TestPrebuildLyx`
- `cmd/testtiming#runResources`
- `cmd/testtiming#measurePackageResources`

**Intent:**
`cmd/testtiming` measures the disjoint tiers (D5).

Its untagged default mode runs `-tags tier0`, so it measures the `tier0` tests once they are tagged; `-full` maps to `tier1`, and the refusal of `-full` beside `-tags` names `tier1`.
The `tier0` default reaches the `go test` arguments only: `resolveTags` still returns `""` for a run with no tags of its own, so `run`'s tier label, its real-git hint and `prebuildLyx` still treat that run as the offline tier and build no `lyx`, and `redundancyTags` still selects `defaultRedundancyTags`.
The `-resources` mode takes the same default, as the usage comment says `-tags` and `-full` pick its tier as in the timing mode: with no tags of its own, `runResources` hands `tier0` to `listPackages` and `measurePackageResources`, so the `go list` and `go test` arguments carry `-tags tier0`, while its tier label and `prebuildLyx` still see `""`.
`prebuildLyx` builds nothing for an explicit `-tags tier0` either, as no `tier0` test spawns `lyx`, and its doc says so; `TestPrebuildLyx` gains a `tier0` row that builds nothing and strips the inherited binary.
`fileTier`, the spawn scan's tag classification, reads `tier1`, `tier2` and `tier3` beside `integration`, `tmux` and `llm`: a file naming `tier2` is tmux-tier, and a file naming only `tier0` counts as untagged.
The `-redundancy` mode's default, `defaultRedundancyTags`, becomes `tier0,tier1,tier2,integration,tmux`, every tier but `llm` under both spellings, so it still compiles every tier's tests but `llm`'s after the retag.
`TestResolveTags` asserts the new mapping, `TestRedundancyTags` the new default, and `TestScanSpawns` gains rows for a `tier2` file and a `tier0` file.
The tool's usage comment and `docs/benchmarks/running-tests.md` name the tier tags and say `tier1` no longer runs `tier0`'s tests.

**ImpactSummary:** Changes the timing and resources modes' default and `-full` tags, the prebuild's skip of a `tier0` run and the spawn scan's classification; no other caller.
