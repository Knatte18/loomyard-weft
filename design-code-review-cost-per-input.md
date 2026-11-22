# Find what costs Code-Burler-Review its extra minutes

`code-review-cost-per-input` · Task · enhancement, burler, lenses · draft

## Seen

`darn-slowdown-measure` (`docs/research/darn-slowdown-measure.md`) found Code-Burler-Review's median up 2.8 min per darn between 2026-11-16/17 and 2026-11-21/22, about 6 % of a 45 min run,
with 0.5 more lens reviews per run.

## Idea

Measure which input of the reviewer costs the minutes, mutation survivors or the tautological-test lens among them.
Cap or skip that input on a diff with no surviving mutant.
