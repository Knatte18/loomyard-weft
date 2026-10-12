# Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss

`twill-plan-followups` · Note · enhancement, loom

## Seen (2026-10-12, from `twill-plan`)

Two items stay open once the `twill-plan` task lands, outside its scope:

- Whether lens review lets Plan-Review run fewer rounds, as the operator chose for Discussion: once twill runs for Plan-Write, measure runs with a lower `plan_max_bounces`.
  The operator hopes twill lets Plan-Review skip one round on average, which would pay for its cost (about $5 per plan against $2.6 for a single `opus[high]` writer).
  The count of review rounds per segment moved into task `convergence-plan` (2026-10-12), whose report answers this.
- A miss every twill-plan bench arm shared: a check against the wrong parent directory; no lens or check covers it yet.
  Records in `.scratch/twill-plan-bench/` in the hub prime.

## First numbers (2026-10-12)

An efficiency audit of the twill planner is wanted later (operator, 2026-10-12).
Two counts taken today belong to it.

Token use by role over the six newest runs, all planned with twill (`lyx tokencount`, report `.scratch/token-usage-by-role.md` in the hub prime):

- The plan writer's own session dominates, not the four twill forks: 0.7M to 9.9M weight per run against 0.3M to 1.4M for the forks.
  Its cost grows with session length: average cache read per message ran from about 88k (`cleanup-1012`, 42 messages) to about 346k (`lyx-halt`, 236 messages).
- Burler fixer use was lower than in earlier reports; whether twill's plans cause that is unmeasured.

Plan-gate failures (`lyx loom validate-plan`) in the plan writer's session:

- With twill: three of six runs failed the gate once, each fixed in one turn, 1 to 3 % of the session's cache read.
- Before twill (2026-10-11 to 10-12 08:20Z, 23 sessions): 13 failed at least once, typically 2 to 4 turns lost, with a tail of 15 turns (`parent-review-wait`, 21 % of its cache read) and 22 turns (`burler-split-budget`, 19 %).
  Which pre-twill sessions were full Plan-Write steps was inferred from plan-file writes, so treat these as approximate.

Gate failures are not what makes the plan writer expensive; the long session is.
