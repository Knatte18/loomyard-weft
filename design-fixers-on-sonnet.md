# Plan and Discussion fixers on Sonnet, once findings are concrete enough

`fixers-on-sonnet` · Note · enhancement, burler, claude

## Seen

`docs/research/token-audit-2026-10-22`: Opus and Sonnet read cache at the same price, so a swap saves about half a role's non-cache-read cost.
Over the audit window the Plan and Discussion fixers cost about $162 on Opus, the darn writer $45, alignment $40.
The Webster and Merge fixers already run on Sonnet: the template sets `webster_fixer_model` and `merge_fixer_model` to `sonnet[medium]` since `c2d13e5ae` (2026-10-21), so the audit's $37 Opus Webster fixer spend predates that.
`discussion_fixer_model` and `plan_fixer_model` are empty and fall back to `fixer_model`, `opus[medium]`.

## Operator view, not a decision (2026-10-22)

A Sonnet fixer needs findings concrete enough for Sonnet to carry out.
For the Webster perch Sonnet is the natural fit, since Webster's own forks are Sonnet.

## Idea (orch, not decided)

Before the Plan and Discussion fixers move to Sonnet, check that their findings are concrete: each names the place and the change, not only the problem.
A3 (`review-setup-audit`) notes a sloppy fixer causes many late-round defects, and `docs/research/model-downgrade-results.md` holds earlier downgrade results; bench before switching.

## Review-setup audit (landed 2026-10-28)

`docs/research/review-setup-audit-2026-10-28/report.md` is the A3 audit named above.
It does not measure fixer quality or a model swap.
Its finding that bears here: in Plan round 3, 14 of 23 findings point at cards the round-2 fixer left alone, so those late defects are the writer's repository-invariant misses and not a sloppy fixer's; the audit recommends `validate-plan` checks for them.

## Machine checks landed (task `validator-checks`, 2026-10-28)

`lym loom validate-discussion` and `validate-plan`, and the gates that call them, now refuse the recurring findings a machine can see: decision ids and acceptance coverage, missing cited paths and symbols, and in plans `seam-test-unlisted`, `test-tier-unstated` and `doc-go-unlisted`.
That is the audit's recommended `validate-plan` step, so the writer's repository-invariant misses are caught before a review round; the fixer-quality question here is unchanged and still needs a bench.
