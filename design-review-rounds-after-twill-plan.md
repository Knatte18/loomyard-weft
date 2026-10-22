# Revisit late review rounds after the twill-plan lens change

`review-rounds-after-twill-plan` · Note · audit, burler, loom

## Seen

The audit `twill-writer-quality-audit` found discussion-review round 3 and plan-review round 2 useful only when an earlier round's fix was too narrow.
Since `504c5552` (2026-10-20), the Plan writer's `twill-plan` fan runs every `review-plan` lens, as the Discussion writer's fan already runs every `review-discussion` lens, which should cut round-1 findings further.

## Idea

Once 4-5 loom runs have landed after `504c5552`, compare their round-1 Plan findings against the audit's baseline, and check whether discussion-review round 3 and plan-review round 2 still find anything.
If they rarely do, lower the default review bounce caps for those segments.
