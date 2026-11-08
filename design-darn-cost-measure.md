# Measure darn cost per run and per 1000 lines, and a darn family against a tapestry

`darn-cost-measure` · Task · research, loom · running · Darn-Write

## Why (operator, 2026-11-08)

The coming `arras` recipe splits a large change into a family of darns instead of one tapestry.
Whether that is cheaper depends on what a darn costs: each darn runs an Opus writer, its own Discussion and its own Code-Review perch, while a tapestry plans once with Opus and implements its cards with Sonnet.
`docs/research/loom/writer-cost-remeasure-2026-11-08/report.md` priced the tapestry side; this prices the darn side.

## Scope

No agent run, no test and no gate run: this reads run records, transcripts, `lym report` output and git history only.
Inputs: every darn run landed from 2026-10-25 to the start of this run, and every tapestry run landed in the same span.

## Measure

Per darn run, at API list price:

- Cost of Discussion (with its review), writer, Code-Review perch (reviewers, judge, fixer), gate fixer and anything else, each on its own line.
- Wall time from `lym report timing`, Code-Review rounds, and changed lines from the landing commit.
- Group by profile (`docs`, `light`, `solid`, others) and give per-run and per-1000-changed-lines figures.

Then:

- The darns of the fabricengine rewrite, filed by one raddle split, summed as one family, against the tapestry runs of the same span, normalised per 1000 changed lines, with each phase's share.
- Which of writer and perch is the larger cost in a darn, and what the writer would cost at Sonnet list price for the same tokens, stated as an estimate only.

## Report

Write `docs/research/loom/darn-cost-2026-11-08/report.md`: per-run tables, the sample size stated, the family-versus-tapestry comparison, and what the figures say for `arras` filing darns.
Commit the report and no code.
