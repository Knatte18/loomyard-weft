# Bench reviewers that name the place and the change in every finding

`concrete-review-findings` · Task · research, burler, stencil · running · Record-Write

## Seen

A cheaper fixer model needs findings concrete enough to carry out: each names the place and the change, not only the problem (operator, 2026-10-22).
Plan and Discussion reviewers do not write findings that way today.
Asking them to costs the reviewer more output and may change what it finds.

## Bench (operator, 2026-11-03)

Runs as darn with profile `docs`, before bench `fixers-on-sonnet`, which takes its concrete findings as input.

- Inputs: the same two artifacts `fixers-on-sonnet` uses, one plan and one discussion from past runs, each at the state its review round saw.
- Arms, each on both inputs, with the reviewer model the hub uses today:
  1. The review stencil as it is, the baseline.
  2. The review stencil with a rule that every finding names its place and the change to make.
- Repeats: two per arm and input, so eight review runs.
- Scoring: findings per run, the share that name place and change, the findings one arm raises that the other misses, and the cost of each run.
- Report under `docs/research/`, keeping the arm-2 findings as `fixers-on-sonnet`'s input, with a recommendation on whether the rule goes into the stencils.
