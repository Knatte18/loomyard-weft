# Re-measure writer exploration and the carry of lookup results after the Discussion stencil change

`writer-lookups-remeasure` · Note · research, loom, stencil

## Idea (operator and orch, 2026-11-04)

Re-run the writer-lookups measurement (`docs/research/turns/writer-lookups-measure-2026-11-04/`, `run-all.sh`) once at least 10 tapestry runs have landed after the Discussion-Write stencil change of `discussion-explore-agents-first`, on those runs, with no new agent run.

- Count, per Plan-Write and Discussion-Write session, whether it started lookup subagents up front, and its exploration turns, chains and cost, as the 2026-11-04 report did.
- Price the carry of lookup results (the context a lookup adds, paid in every later turn) as a saving of its own: compare the carry of sessions that delegated early against those that did not, which the 2026-11-04 report measured but left out of its savings estimate.
- Cap: the 10 most recent tapestry runs landed after that change; both writer roles of each.
- Runs as darn with profile `docs`.

## Built since (tidy, 2026-11-05)

Task `discussion-explore-agents-first` landed (`3085f451`): the Discussion-Write stencil now opens Step 2 with one batch of explore subagents.
Runs landed after that commit count toward the 10.
