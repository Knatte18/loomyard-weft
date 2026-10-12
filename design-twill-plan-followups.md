# Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss

`twill-plan-followups` · Note · enhancement, loom

## Seen (2026-10-12, from `twill-plan`)

Two items stay open once the `twill-plan` task lands, outside its scope:

- Whether lens review lets Plan-Review run fewer rounds, as the operator chose for Discussion: once twill runs for Plan-Write, measure runs with a lower `plan_max_bounces`.
  The operator hopes twill lets Plan-Review skip one round on average, which would pay for its cost (about $5 per plan against $2.6 for a single `opus[high]` writer).
- A miss every twill-plan bench arm shared: a check against the wrong parent directory; no lens or check covers it yet.
  Records in `.scratch/twill-plan-bench/` in the hub prime.
