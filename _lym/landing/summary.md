# Measure darn cost by phase and a darn family against tapestry runs

Adds `docs/research/loom/darn-cost-2026-11-08/`: a report, the scripts that produce its tables and the tables themselves.
It changes no code.
The report prices what a darn costs per run and per 1000 changed lines, by phase and by profile,
and sets the 12 darns of the `fabric-test-speed` raddle split against the tapestry runs of the same span.

## What the report finds

- A darn costs $5.45 on average over 284 landed runs, $4.63 at the median, and takes 38 awake minutes.
  Cost is mostly per run: a fit on changed lines gives $4.48 a run plus $1.43 per 1000 lines, r 0.31.
- The writer is the larger cost, 58% of a darn against 37% for the Code-Review perch.
  At Sonnet list price for the same tokens, an Opus writer session costs 63% as much; this is an estimate only.
- The 12 family darns cost $39.88, and $58.53 with the Motif Discussion run and the raddle run before them, for 4,134 lines.
  A fit of 15 tapestry runs puts one tapestry of that size at $60.08.
  The split saves the Plan phase and pays for it in Code-Review: 28% of the family's cost against 4% of a tapestry's.
- No darn in the sample ran a Discussion agent; the decision record is written by Go or comes from a Motif run.

## Design decisions

- Costs come from `tools/tokencount` transcripts, so they cover all 291 darn runs;
  the usage records hold profile, stored-record flag and landed lines only from 2026-11-05 on, so 208 darns form an unlabelled group.
  Changed lines come from each run's landing commit and equal the usage record's figure on the 79 runs that have one.
- The report leaves `comment-cleanup` out of the means: its comment-lead seat cost $335 and it is not a normal darn.
  It leaves five darns of more than 10,000 changed lines out of the per-1000-line and fit figures.
  `cold-gate-fixer`, a tapestry run with no Discussion, Plan or Webster transcripts on this machine, is left out of the tapestry side.
- The task's "fabricengine rewrite" family is read as the 12 parts of the `fabric-test-speed` raddle proposal,
  whose project is the `internal/fabricengine` tier 1 time.
- The scripts and tables sit beside the report, as the earlier writer-cost remeasure does, so each figure can be rerun.
- No `PATTERN.md` entry changes, and the decision record lists none.

## Verification

No agent run, test or gate run was made, as the task asks.
`tables.sh`, `family.sh` and `sonnet-writer.sh` regenerate every table of the report from the saved tables.
The line counts of `run-table.tsv` were checked against the usage records' `landed.json` on the 79 runs that hold one.
