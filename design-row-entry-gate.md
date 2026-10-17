# Entry gate on shed rows: a check decides whether a row runs at all

`row-entry-gate` · Note · enhancement, shed, loom, batten

## Seen (orch, 2026-10-17)

Two ways exist today to leave out a row that is not wanted:

- Darn turns its review perch off by a recipe rewrite: with `review_max_bounces` 0, `internal/loomrecipe` rewrites Darn-Write's Done edge straight to Merge-In when the recipe loads (`contracts/recipes/darn-recipe.yaml`), so the perch rows are never entered.
- Batten's Open-Window, Seed-IDE and Open-IDE rows are `shedadapters`' switched producer: the row reads its `batten.yaml` flag and is Done without acting when the flag is off.

## Operator view, not a decision (2026-10-17)

The recipe rewrite is a terrible solution.
A Bouncer should instead have an entry gate: a check that runs first and decides whether the Bouncer runs at all.
For darn's perch that check is only whether `review_max_bounces` is above 0, and the producer that opens VS Code works the same way: it checks its flag and skips everything when the flag is off.

## Orch recommendation, not a decision

- The entry gate belongs in shed, for every row, since every FSM needs it: a row in the recipe may name an entry check, read from the run's config stack when the row is reached; when it is false, the row is Done without acting and the run follows its Done edge.
  A perch's Burler row is reached only through the Bouncer's stuck edge, so a skipped Bouncer skips the whole perch.
- Status and the run's history record the row as skipped with the check's reason, so a skip is visible.
- The switched producer's flag check becomes this gate, and `internal/loomrecipe`'s edge rewrite for darn is removed.
- The regression perch (note `regression-baseline-verify`) and the `Command` rows (note `command-producer`) use the same gate.

## Open

- The form of a check: a config key that must be true or above 0, or a small fixed set of predicates; never free code.
- Whether a task's `run_settings` can switch a gate per task: it can today for darn's budget and batten's flags, since both read the pair's config stack.
