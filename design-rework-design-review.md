loom-recipe.yaml:37 exempts PR-Rework from Plan-Review. Keep the exemption for text-only findings; route cards that create or change a classifier, verb or guard through Plan-Review.

## Rework should not drag the implemented plan along (operator, 2026-10-01)

PR-Rework appends its cards to the same plan, and Webster resumes the same run, so Master's prompt and context carry every earlier card's index and progress line.
In refusal-audit round 4 that was 64 batches with 54 done; the rendered prompt outgrew the launch limit (fixed on main in `e39909bbc` by handing Master the prompt as a file, which treats the symptom only).
Done cards are dead weight: Master never re-runs them, but they fill its context and every fork inherits it.
Want: each rework round is its own plan generation; Webster sees only that round's cards plus `00-overview.md`, and the earlier cards are archived (git keeps them), so prompt and context stay flat however many rounds a task takes.
