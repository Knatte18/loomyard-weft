# Audit the review setup: rounds per perch, lens worth, finding classes

Adds `docs/research/review-setup-audit-2026-10-28/report.md`, an audit of the review setup read from the records of the loom runs landed since `79bab47a0`.
It changes no code, configuration, profile, lens set or stencil.

## What the report covers

- 50 archive tags hold per-lens reports in the weft repository; 8 of them are full loom runs with Discussion and Plan perches, the rest hold code review only.
- Rounds per perch: findings, gating findings, fresh tokens and minutes per round, how many findings stay fixed, and whether Discussion round 3 and Plan rounds 2 and 3 still find defects (they do: gating findings in 6 of 7, 7 of 8 and 4 of 4 runs).
- Lenses: per lens for `twill-plan`, `twill-discussion`, `review-discussion`, `review-plan` and `review-code`, with yield, noise and overlap.
- Finding classes at Discussion and Plan, with the cheapest place to stop each.
- Fix-delta review: how many later-round findings point at text the previous fixer changed.
- Each question ends in a recommendation marked as a recommendation, and every number carries its sample size.

## What sits beside the report

The extraction scripts (`extract-*.sh`, `classify-findings.sh`, `lens-hit-rate.sh`, `unpack-runs.sh`), the tag list `runs.txt` and the TSV tables they produced.
Every figure in the report derives from them.

## Design decisions

- Token and minute figures come from each round's `round-<n>-usage.yaml`, as fresh tokens without cache reads.
  `lym report convergence` and `lym report timing` refuse outside the prime worktree, and `tools/tokencount/tokencount.sh` runs through `go run`, so none of the three ran; the report says so.
- Real-or-noise is the writer's judgement by stated rules: a kept review finding counts as real, a fix holds when the next round's ledger lists the key as resolved, and a writer-side lens item counts by the writer's own `fixed` or `rejected` disposition.
- Finding themes and fix-delta touching are keyword heuristics, and the report labels them so.
- The 2026-10-22 baseline's proposal to fold `code-affected-set` into `code-pattern-fit` is reversed in the recommendation, because `code-affected-set` found a gating finding in 8 of 58 rounds against 3 for `code-pattern-fit`.
