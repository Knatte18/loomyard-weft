## Goal

```markdown
## Seen

`docs/research/token-audit-2026-10-22`: Opus and Sonnet read cache at the same price, so a swap saves about half a role's non-cache-read cost.
Over the audit window the Plan and Discussion fixers cost about $162 on Opus.
The Webster and Merge fixers already run on Sonnet (`webster_fixer_model` and `merge_fixer_model` set to `sonnet[medium]`).
`discussion_fixer_model` and `plan_fixer_model` are empty and fall back to `fixer_model`, `opus[medium]`.
A Sonnet fixer needs findings concrete enough to carry out: each names the place and the change, not only the problem (operator, 2026-10-22).
`docs/research/review-setup-audit-2026-10-28/report.md` found the late Plan-round defects to be the writer's misses rather than a sloppy fixer's, and `validator-checks` now catches the recurring machine-visible ones; fixer quality on a swap is still unmeasured.

## Bench (operator, 2026-11-03)

Runs as darn with profile `docs`, after bench `concrete-review-findings` has landed, since its third arm takes that bench's concrete findings as input.

- Inputs: two review rounds from past runs with their findings, one Plan round and one Discussion round.
- Arms, each on both inputs:
  1. `opus[medium]` fixer on the findings as written today, the baseline.
  2. `sonnet[medium]` fixer on the findings as written today.
  3. `sonnet[medium]` fixer on the same rounds' findings written in the concrete form.
- Repeats: two per arm and input, so twelve fixer runs.
- Scoring: a fresh review round of each fixed artifact counts the findings fixed correctly, left open and newly made; the cost of each fixer run is recorded.
- Report under `docs/research/`, with a recommendation for `plan_fixer_model` and `discussion_fixer_model`.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-03): Drop arm 3; score with a solo reviewer

Decision: Drop arm 3: run arms 1 (opus[medium]) and 2 (sonnet[medium]) on the two rounds in concrete-review-findings' data/fixers-input.tsv, two repeats each, eight fixer runs. Score each fixed artifact with an in-session solo reviewer, the same reviewer model and prompt for every run.

Rationale: The reviewer bench found every finding already names place and change, so arm 3 has no input distinct from arm 2; a solo reviewer applied alike to every arm keeps the scoring fair though the hub's fork fan cannot be spawned here.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
