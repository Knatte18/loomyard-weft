# Burler: spawn the chair only when the review is ready

`burler-chair-late-spawn` · Note · enhancement, burler

## Seen (2026-10-11)

In the four newest loom runs (`.scratch/token-usage-by-role.md` from tools/tokencount), the Burler chair has the longest turn time of any role (22–30 min per run, 35–39% of all turn time) while costing less than the advisor.
Operator, 2026-10-11: the chair is spawned first and waits for the reviewer; that wait is likely cache waste, since the chair's prompt cache can expire before the review arrives.

## Idea

Spawn the chair only when the review is ready, so it reads the review on a warm start and holds no turn open while it waits.

An analysis agent is measuring the wait per round, whether the chair's cache expired in it, and the cost of the re-write (`.scratch/burler-cost-breakdown.md`, 2026-10-11).
