# Drop the transitional tag spellings

Every test file now carries one `tierN` tag, and the deployed `lyx` runs the tier-named guard scan.
That leaves no reason to keep the old `integration`, `tmux` and `llm` spellings beside the tier tags, so this change removes them from build constraints, package loads and guards.

## What changes

- **Guard files.**
  `//go:build tierN || integration` becomes `//go:build tierN` on every guard-marked file.
- **One-tag scan.**
  `cmd/lyx/tiertags_test.go` drops the `|| integration` allowance.
  It now refuses any test file whose constraint names `integration`, `tmux` or `llm`, guard-marked or not, and names the `tierN` tag to use instead.
- **Package loads.**
  `internal/impactset`'s two `go list` calls and `internal/planglyph`'s `packages.Load` run under `tier0,tier1,tier2,tier3` only.
  `impactset`'s test row for a transitional `tier1 || integration` guard file is gone.
- **`cmd/testtiming`.**
  The spawn scan reads the tmux tier from `tier2` alone, and `defaultRedundancyTags` is `tier0,tier1,tier2`.
- **Raw `go test` guards.**
  `internal/shellguard`'s raw go row and `internal/planparser`'s `verify-module-wide` check refuse a `go test` that selects `tier2` or `tier3`, and no longer name `tmux` or `llm`.
  `claudeengine`'s settings golden follows the row.
- **Tier guards.**
  `knownTierTags` and `isTmuxTier` in the tier purity guard, `isolationTags` and `isolationTagSets` in the tmux isolation guard, and the LLM tier guard name tier tags only.
  Their fixture rows are respelled, and the expected failure counts follow the smaller tag-set list.
- **Fixtures.**
  `internal/testkit/tmuxkit/allow_tmux.go` builds under `tier2 || tier3`, and `internal/planglyph/testdata/glyphchain/shapes/tagged.go` builds under `tier1`.
- **Docs.**
  `internal/gatecli`'s example tier commands, and `impactset`'s and `planglyph`'s package docs, drop the old spellings.
- **Hub tier commands.**
  The hub's `gate.yaml` on Bolt now names one tag per tier command, committed by the orch as `28da14696`, because a task session may not write hub-wide config.

## Design decisions to know

- The one-tag scan refuses an old spelling outright rather than ignoring it.
  No tier command sets those tags any more, so a file still carrying one would silently drop out of every tier.
- The LLM tier guard's `llm` allowance goes too, even though the board entry does not list it, because otherwise it would accept a spelling the one-tag scan refuses.
- Prose comments that mention `-tags integration` and similar in files this change does not otherwise touch are left alone.

## To check by hand

- Right after the hub's tier commands changed, tier 1 failed for a while before any test ran.
  `go vet` read `internal/reedengine/proctree_linux.go` from the torn-down `gate-runs-once` worktree through a stale build-cache entry.
  The error has since stopped, and tiers 0, 1 and 2 of the `darn-write` gate pass on this tree under the new commands.
  If another branch hits the same error, the machine's Go build cache is the first suspect.
- `docs/benchmarks/test-redundancy.md` still records an old `integration,tmux` run.
  A default `-redundancy` run now asks for `tier0,tier1,tier2` and refuses to splice into it until the report is regenerated whole.
