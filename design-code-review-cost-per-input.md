# Find what costs Code-Burler-Review its extra minutes

`code-review-cost-per-input` · Task · research, burler, lenses · starting · Seed-Child

## Seen

`darn-slowdown-measure` (`docs/research/darn-slowdown-measure.md`) found Code-Burler-Review's median up 2.8 min per darn between 2026-11-16/17 and 2026-11-21/22, about 6 % of a 45 min run,
with 0.5 more lens reviews per run.

## Goal

Find which input or step of Code-Burler-Review costs the extra minutes, mutation survivors and the tautological-test lens among the suspects.

## Bounds

- Read archived run records (`refs/tags/archive/<slug>/<sha>` in `loomyard-weft`) and git history only: about 15 darn runs from each period, the same periods as the earlier report.
- No timed runs and no agent runs.
- The report ends with each cost, its share, and a concrete fix for each as a candidate note, such as capping or skipping an input on a diff with no surviving mutant.
