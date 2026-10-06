# Card 3 — tooling-contracts-prune

**Edit:**
- `cmd/testtiming#`
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
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of `cmd/testtiming`, `contracts/specs`, `contracts/stencils` and every `tools/...` package.

`cmd/testtiming`'s behavior stays as `test-suite-measure` built it; only its tests are pruned or folded, and no production file there changes.
`contracts/stencils` holds most candidates of this group: its stencil-content checks are fold leads, and a check that pins a stencil's exact text is a unique assertion under Decision 2.
`tools/sandbox` keeps exercising, or explicitly excluding with a reason, every lyx module (`PATTERN-sandbox-coverage`): a fold or deletion there keeps that check intact.

**ImpactSummary:** Test files of these packages only; no production change, and `cmd/testtiming`'s behavior is untouched.

**Commit:** 3: Prune tool and contract tests
