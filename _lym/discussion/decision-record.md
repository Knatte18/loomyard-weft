## Goal

```markdown
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
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-03): Audit before any review run

Decision: Take option (b): run no review bench first. Audit the Plan and Discussion review findings on the weft's recent runs for the share whose fix key names both place and change, and report it with the stencil's existing rule (burler-review-format.md since add9f93d). Run option (a), a stricter rule on top, as the eight solo-reviewer runs, only if the audit shows real non-compliance; otherwise the report says so and keeps a set of compliant findings from the two chosen rounds for bench fixers-on-sonnet to use.

Rationale: The premise is overtaken: the stencil already demands concrete fix keys, so the bench as written would mostly compare noise at a real run cost.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
