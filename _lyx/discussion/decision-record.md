# Decision record — drop-transitional-tag-spellings

## Goal

Every test file now carries one `tierN` tag and the deployed `lyx` runs the tier-named guard scan, so the old `integration`, `tmux` and `llm` spellings kept beside the tier tags go.
After the change no build constraint, tier command, package load or guard in the repository names an old spelling as a tier.

## Scope

- Guard files: `//go:build tierN || integration` becomes `//go:build tierN`.
- `cmd/lyx/tiertags_test.go`: the `|| integration` allowance on guard-marked files goes.
- The hub's tier commands in the hub-wide `gate.yaml`: `-tags tier1,integration`, `tier2,tmux` and `tier3,llm` become the single tier tag.
- `internal/gatecli`'s doc example of the tier commands follows the hub file.
- Package loads in `internal/impactset` and `internal/planglyph` load `tier0,tier1,tier2,tier3` only.
- `cmd/testtiming`: the spawn scan's tier reading and `defaultRedundancyTags` name tier tags only.
- The two guards refusing a raw `go test` of `tier2` or `tier3`: `internal/shellguard`'s raw go row and `internal/planparser`'s verify-module-wide check stop naming `tmux` and `llm`.
- The tier purity guard's `knownTierTags` and `isTmuxTier`, and the tmux isolation guard's `isolationTags` and `isolationTagSets`, name tier tags only.
- `internal/testkit/tmuxkit/allow_tmux.go`'s constraint becomes `tier2 || tier3`.
- `internal/planglyph/testdata/glyphchain/shapes/tagged.go` is tagged `tier1`.
- Each guard's fixture rows and doc comments follow the new spellings.

## Decisions

### The one-tag scan refuses an old spelling outright

Decision: `cmd/lyx/tiertags_test.go` fails any test file whose constraint names `integration`, `tmux` or `llm`, guard-marked or not, naming the tag and the `tierN` tag to use.
Rationale: an old spelling left in a constraint after this task is a file the tier commands no longer select, so it is a silent test loss; a branch cut before the retag lands with such files and needs the loud refusal.

### The LLM tier guard drops its `llm` allowance too

Decision: `cmd/lyx/llmtier_test.go` confines llmkit imports to `tier3` alone, and its fixture rows use tier tags.
Rationale: it is a guard holding an old spelling beside the tier tag, the same kind of allowance the board entry removes from the other guards; the one-tag scan's refusal would otherwise contradict its message.

### The hub's tier commands are written through `lyx config gate`

Decision: the hub-wide `gate.yaml` tier commands are changed through `lyx config gate --set`, never by hand-editing the board dir.
Rationale: `PATTERN-hub-wide-config` puts the file on the board and `lyx config` is its writer and committer.

### Comments outside the touched files keep their old spellings

Decision: prose mentions of `-tags integration` and similar in files this task does not otherwise touch stay as they are.
Rationale: the board entry names the code sites; a repository-wide comment sweep is a separate change and adds review noise.

## Constraints

- `PATTERN-tier-tags`: every test file carries exactly one `tierN` tag.
- `PATTERN-hub-wide-config`: the hub's `gate.yaml` lives on the board and is written through `lyx config`.
- `PATTERN-shell-guard-table`: the raw go row lives in `internal/shellguard` and every engine renders it, so `claudeengine`'s settings golden follows the pattern.
- `PATTERN-test-economy`: fixture rows are respelled or dropped in place, never duplicated.

## Auto-mode assumptions

- The deploy the board entry waits on has happened: the hub's `gate.yaml` was last reconciled by the production build of `ed98ccf71`, the commit that brought the tier tags.

## Open risks

- A task branch cut before `ed98ccf71` that still holds `integration`, `tmux` or `llm` test files runs none of them under the new tier commands until it merges `main`; the one-tag scan then fails it loudly.
- The hub's `gate.yaml` change takes effect for every run of the hub when written, before this branch lands.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - pattern-test-isolation-retired-tag-sets (class consistency, severity MEDIUM, open in rounds 1)
  - pattern-test-speed-transitional-guard-tag (class consistency, severity MEDIUM, open in rounds 1)
  - retired-spelling-scan-misses-right-of-false-and (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- No `//go:build` line in the repository names `integration`, `tmux` or `llm`.
- The one-tag scan fails a test file whose constraint names an old spelling, guard-marked or not, and passes a guard file tagged `tier0` alone.
- The tier purity, tmux isolation and LLM tier guards classify `tier1`, `tier2` and `tier3` files as before and no longer treat an old spelling as a tier.
- The raw go shell row and the plan's verify-module-wide check refuse `go test -tags tier2` and `-tags tier3`, and no longer name `tmux` or `llm`.
- `cmd/testtiming`'s default redundancy tags are `tier0,tier1,tier2`, and its spawn scan reads the tmux tier from `tier2` alone.
- `impactset` and `planglyph` load packages under `tier0,tier1,tier2,tier3`, and the glyphchain `tagged.go` member resolves under `tier1`.
- The hub's tier commands name one tier tag each.
