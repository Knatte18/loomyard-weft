# Card 3 — tooling-contracts-prune

**Edit:**
- `contracts/specs#`
- `contracts/stencils#`
- `tools/codestats#`
- `tools/deploy#`
- `tools/godocreflow#`
- `tools/internal/devbin#`
- `tools/mdreflow#`
- `tools/sandbox#`
- `tools/wordswap#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `contracts/specs`, `contracts/stencils` and every `tools/...` package.

This card edits no file under `cmd/testtiming`, tests included: `test-suite-prune-core` changes that package's redundancy mode and its tests.
`contracts/stencils` holds most candidates of this group: its stencil-content checks are fold leads, and a check that pins a stencil's exact text is a unique assertion under Decision 2.
`tools/sandbox` keeps exercising, or explicitly excluding with a reason, every lyx module (`PATTERN-sandbox-coverage`): a fold or deletion there keeps that check intact.

**ImpactSummary:** Test files of `contracts/specs`, `contracts/stencils` and the `tools/...` packages only; no production change, and nothing under `cmd/testtiming`.

**Commit:** 3: Prune tool and contract tests
