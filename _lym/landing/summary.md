# Re-measure writer lookups and Plan phase cost on five tapestry runs

Adds `docs/research/loom/writer-cost-remeasure-2026-11-08/` and one line in `docs/research/README.md`.
No code changes.

The report reads the five tapestry runs that landed after `3085f451`
(`pawl-orch-landing`, `raddle-recipe`, `drift-rework`, `slack-relay`, `creel-cross-hub-build`) from their transcripts and records.
No agent, test or gate run was made.

## What the report finds

- Writer lookups: all 3 Discussion-Write sessions now send one batch of lookup subagents at turn 6 or 7, where 1 of 3 did before;
  4 of 5 Plan-Write sessions send theirs by turn 12.
  In Discussion the carry of lookup results fell from $1.37 to $0.84 a session against the two sessions that did not delegate,
  and the subagents cost $0.80, so the session cost is unchanged ($3.05 against $3.06).
  Priced as a saving of its own, the lookup context kept out of the writers is worth at most $5.95 of $46.94 over the 8 sessions.
- Plan phase: per run it has not fallen (writer plus forks $17.05 against $17.24 in the audit's comparable runs),
  but the sample's tasks are 2.9 times as large by changed lines,
  so writer plus forks costs $1.73 per 1000 lines against $5.11 and $0.84 per plan card against $1.21.
  Plan rounds rose from 2.2 to 3.2, and review and fix are 51% of the phase.
- Each open idea of the board note `plan-writer-context` is checked against these figures in a table:
  the figures support the review-cost and excerpt ideas as the larger pools, give no cost case for sub-planners,
  and carry no solo-writer arm.

## Design decisions

- The Plan phase comparison with the 2026-10-24 audit uses that audit's committed costs,
  because its runs have no transcripts left on this machine; only their sizes are looked up again (`audit-sizes.sh`).
  Two of its eleven runs have no findable landing commit and are left out of the normalised rows.
- The report adds the 10 tapestry runs between the audit and the sample as a middle group, and a size-band comparison and fits,
  since per-1000-line averages fall with size partly through a fixed cost per run.
- Plan-Write wall time comes from `tools/tokencount`'s producer table;
  `lym report timing` reports the whole Plan phase per run and has no per-producer split, so its figures stand beside as `timing-plan-phase.tsv`.
- The writer-lookups scripts are copied into the new directory with the sample and cutoff changed, not edited in place,
  so the 2026-11-04 tables stay as they were.
  New scripts price the carry (`carry.sh`), the context each lookup subagent read (`lookup-context.sh`),
  the Plan writer's own reads by call kind (`plan-reads.sh`), and the Plan phase cost and size tables (`plan-table.sh`, `audit-sizes.sh`, `group-means.sh`).

## Review notes

- The tables and the report are one run on one machine; every rate in them is a case count over 5 runs.
- The carry-avoided figure is an upper bound: it assumes the writer would have kept everything a subagent read.
- Subagent and fork costs in `writer-subagents.tsv` are priced by the script and run about 13% above `tools/tokencount`;
  the report uses `tools/tokencount` for run totals.
