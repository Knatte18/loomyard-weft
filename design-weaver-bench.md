# Bench Weaver against Webster on plans Webster already implemented: time, cost and quality

`weaver-bench` · Task · research, weaver, webster · starting · Open-Window

Depends on: [`bench-makes-own-pairs`](design-bench-makes-own-pairs.md)

## Aim (operator, 2026-11-27)

Weaver, the cold-start implementer with model escalation (`18abfaf50`), landed long ago and has never been tried on a real plan.
This bench runs it on plans Webster has already implemented and landed, and measures time, cost and quality against Webster on the same plans.
It is large, long and costly, and the operator judges it worth that.

## Plans

- Only existing plans that Webster implemented and that landed; no new plans are written.
- Three plans, found in the records history (the plan directory and `_lym/webster/` state of landed tapestry runs), chosen so that:
  - each is a Go plan of the current plan format, with 4 to 12 cards and at least two batches in Webster's partition;
  - the three touch different packages;
  - each one's base commit, the task branch's commit the plan was implemented on, can still be checked out and builds.
- The report names each chosen plan, its landed commit and why it was chosen.

## Runs

- Each plan is implemented once by Weaver, from the plan's base commit, with the current lym and the hub's default settings: three runs.
- One of the three plans, the one whose recorded Webster run is newest, is also implemented once by Webster under the same conditions, as a calibration: four runs in all (operator, 2026-11-27).
  The calibration shows how far Webster's recorded numbers lie from today's lym, machine and load,
  and the other two plans compare Weaver with their recorded Webster runs, adjusted by that calibration, with the adjustment stated.

- Weaver uses its default ladder (`lym config weaver`); Webster its default engine and models.
- Each run has a bench pair of its own, which the parent creates from this task's worktree when the run asks for it; this task's agent runs them.
  The calibration plan's two runs run at the same time so they share the machine's load.
- No run is repeated; a run lost to the machine (a reboot or an infrastructure stop) is rerun once and the report says so.

## Measures

- Wall time from the implementer row's start to its end, and the time in batches, gates and fixers.
- Cost: tokens and dollars per model, from the sessions' usage records.
- Weaver's ladder: how many attempts climbed past the first rung, and the stops that reached the stop chain.
- Quality:
  - whether the plan-level verify gate passes, and how many fixer rounds it took;
  - the cards left undone or deviated from;
  - a diff of each result against the landed commit, and the findings one Code-Review round (the hub's Burler reviewer, once per run) gives on each result.

## Output

A report under `docs/research/weaver/` with one table per measure across the four runs and the recorded Webster runs, the findings per run, and a recommendation:
whether Weaver can replace Webster as tapestry's implementer, and what it would need first.
