# Re-measure writer lookups and Plan phase cost on the tapestry runs since 3085f451

`writer-cost-remeasure` · Task · research, loom, stencil · running · Finalize

## Why (operator, 2026-11-08)

Two open questions read the same tapestry runs: whether the writers now delegate lookups early and what that saves, and whether the Plan phase has got cheaper.
The operator does not want to wait for 10 runs: take every tapestry run landed after `3085f451` (the Discussion-Write stencil change of `discussion-explore-agents-first`), however many there are.
No agent run, no test and no gate run: this reads records, transcripts and code only.

## Part 1: writer lookups

Re-run the writer-lookups measurement (`docs/research/turns/writer-lookups-measure-2026-11-04/`, `run-all.sh`) on those runs:

- Per Plan-Write and Discussion-Write session: whether it started lookup subagents up front, its exploration turns, chains and cost, as the 2026-11-04 report did.
- Price the carry of lookup results (the context a lookup adds, paid in every later turn) as a saving of its own: compare the carry of sessions that delegated early against those that did not.

## Part 2: Plan phase cost

Per run, at API list price, as `docs/research/loom/plan-writer-audit-2026-10-24/` measured it (rerun per its README where it fits):

- Plan writer alone, its forks, the writer plus forks, the whole Plan phase (writer, forks, review, fix), Plan rounds, and Plan-Write wall time from `lym report timing`.
- Put the figures beside that audit's table (solo writer about $9.08 per run; full twill-plan fan $18.14; five lenses $16.64), and normalise by task size (cards or changed lines) so larger tasks do not read as dearer writing.
- Say whether the Plan phase's cost has fallen since 2026-10-24, and what drove any change.

## Report

Write `docs/research/loom/writer-cost-remeasure-2026-11-08/report.md`: both parts with per-run tables, the sample size stated, and for each open idea in the board note `plan-writer-context` (R6) whether the figures still support it.
Commit the report and no code.
