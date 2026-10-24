# Drop the transitional integration/tmux/llm tag spellings

Every test file already carries one `tierN` tag, and the deployed `lyx` runs the tier-named guard scan.
The old `integration`, `tmux` and `llm` spellings kept beside the tier tags now go.
After this change no build constraint, tier command, package load or guard names an old spelling as a tier.

## What changes

- Guard files: `//go:build tierN || integration` becomes `//go:build tierN`.
- `cmd/lyx/tiertags_test.go`: the one-tag scan no longer allows `|| integration` on guard-marked files.
  It now fails any test file whose constraint names `integration`, `tmux` or `llm`, guard-marked or not, and names the `tierN` tag to use.
  The scan walks the constraint tree, so a retired spelling right of a false term (`tier1 && integration`) is still seen.
- Hub tier commands in the hub-wide `gate.yaml` take one tag each (`tier1`, `tier2`, `tier3`); `internal/gatecli/doc.go` follows.
- `internal/impactset` and `internal/planglyph` load packages under `tier0,tier1,tier2,tier3` only.
- `cmd/testtiming`: the spawn scan reads the tmux tier from `tier2` alone, and `defaultRedundancyTags` is `tier0,tier1,tier2`.
- The two guards refusing a raw `go test -tags tier2|tier3`, the raw go row in `internal/shellguard` and the verify-module-wide check in `internal/planparser`, no longer name `tmux` and `llm`.
  `claudeengine`'s settings golden follows the row.
- The tier purity guard (`knownTierTags`, `isTmuxTier`), the tmux isolation guard (`isolationTags`, `isolationTagSets`) and the LLM tier guard (`llmtier_test.go` confines llmkit imports to `tier3`) name tier tags only; their fixture rows are respelled in place.
- `internal/testkit/tmuxkit/allow_tmux.go`'s constraint becomes `tier2 || tier3`; the messages in `tmuxkit` tell authors to tag `tier2` or `tier3`.
- `internal/planglyph/testdata/glyphchain/shapes/tagged.go` is tagged `tier1`.
- `pattern/PATTERN-test-speed.md`, `PATTERN-test-isolation.md`, `PATTERN-test-economy.md`, `PATTERN-testkit.md` and `contracts/specs/loom-plan-spec.md` drop the transition rule and the old spellings.

## Design decisions

- The one-tag scan refuses an old spelling outright rather than ignoring it.
  A leftover old spelling is a file the tier commands no longer select, so it would be a silent test loss; a branch cut before the retag needs the loud failure.
- The LLM tier guard drops its `llm` allowance as well, the same kind of allowance the other guards lose.
- The hub's tier commands were written through `lyx config gate --set`, per `PATTERN-hub-wide-config`, not by editing the board dir.
- Prose mentions of the old spellings in files this task does not otherwise touch are left alone; a repository-wide comment sweep is a separate change.

## Risks and checks by hand

- A task branch cut before the retag that still holds `integration`, `tmux` or `llm` test files runs none of them under the new tier commands until it merges `main`; the one-tag scan then fails it.
- The `gate.yaml` change applies to every run of the hub as soon as it was written, before this branch lands.
- Check by hand that `tier1 && integration` is refused by the one-tag scan: `cmd/lyx/tiertags_test.go:73`, `firstRetiredTierSpelling` (previously short-circuited past the right operand).
- Check by hand that `pattern/PATTERN-test-speed.md:19` says a guard file takes its one `tierN` tag and that the scan refuses the retired spellings.
- Check by hand that `pattern/PATTERN-test-isolation.md:14` and the lines near 27, 28, 47 and 48 name `tier0` through `tier3` instead of the retired tag sets.
- Check by hand that `contracts/specs/loom-plan-spec.md:506` names the `tier0` through `tier3` tags only for the types load.
