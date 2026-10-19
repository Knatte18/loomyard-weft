# A plan or discussion re-entered for a small amendment gets one review round, not a fresh full budget

`amend-review-budget` · Note · enhancement, loom, burler, shed

## Seen (2026-10-19, run `ubuntu-2404-support`)

The plan had passed three Plan review rounds when Webster halted on card 2.
The fix was a small amendment to that one card, made through `lyx loom goto --to Plan-Write`.
The goto reset the Plan segment's budget, so the amended plan faced the full fresh review budget again (2 rounds, hard cap 3) for a change to one card.
The orch cut it to one round by hand in the pair's `.lyx/config/loom.local.yaml` (`plan_max_bounces: 1`, `plan_hard_max_bounces: 1`), on the operator's order.

## Operator view, not a decision (2026-10-19)

A lightly revised plan does not need three new review rounds.

## Idea

A segment re-entered for an amendment of an already reviewed artifact gets a small review budget by itself, one round by default, configurable in `loom.yaml`, instead of the full fresh budget.
The review of such a round may be scoped to the change against the reviewed artifact.
