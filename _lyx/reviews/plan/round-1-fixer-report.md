# Round 1 fixer report — plan

All seven findings of `round-1-review.md` are fixed.
`lyx loom validate-plan` still reports `ok: true` over the edited plan, with informational findings only.
The `approved:` key in `00-overview.md` is untouched.

## Changed

- **F1** (`09-mark-guards.md`, `08-impact-set-engine.md`):
  card 9 states one coexistence rule for `//lyx:guard` and `//testtiming:keep`: a test carrying both stacks them as contiguous directive lines directly above its func line, in either order, and two stacked keeps stay an error.
  Card 9 adds `cmd/testtiming/keep.go` and `cmd/testtiming/keep_test.go` to its `**Edit:**` list, so `scanKeeps` skips `//lyx:` lines between a keep and the func line, with the two test rows.
  Its `PATTERN-test-speed.md` sentence and `ImpactSummary` follow.
  Card 8's guard parser accepts a `//testtiming:keep` line between the marker and the func line, and `guards_test.go` gains the stacked-marker row.
- **F2** (`11-tmux-tier-green.md`): the false claim that the harness lacks a pinned size is replaced by the facts (the harness pins 140x42 and `window-size manual`, and the subtest passes elsewhere), and the card now has the implementer reproduce the failure under the condition `docs/benchmarks/test-suite-timing.md` records and fix its actual cause.
- **F3** (`07-lint-comments-verb.md`): "takes no arguments" becomes "no positional arguments, and `--base` is its one flag".
- **F4** (`07-lint-comments-verb.md`): states that `loom` stays covered at module level by the core sandbox suite's `**Covers:** loom` scenario, so the verb needs neither a suite row nor an exclusion.
- **F5** (`07-lint-comments-verb.md`, `10-round-gate.md`): card 7 adds `pattern/PATTERN-gate-self-check-parity.md` to its `**Edit:**` list for the card-gate and verb pair, and card 10 adds the file too, for the round gate joining the pair and the Webster-Burler `verify` gate line.
- **F6** (`10-round-gate.md`): the bound now says the round only compiles the `tmux` and `llm` tiers, where a `tmux`-only failure is caught, and that an `llm`-only failure is left to the by-hand run.
- **F7** (`12-publish-verify.md`): the bound now says a landing into a parent outside `require_pr_to_base` runs no `tmux` tier and is gated only by Finalize's plan verify.

## Deferred

Nothing deferred.
