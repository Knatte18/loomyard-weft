# PR-Rework as its own plan generation, design-reviewed

`rework-generations` · Planned · feature · done

## Goal

A PR-Rework round becomes a new plan generation: Webster sees only that round's cards plus `00-overview.md`, earlier cards are archived (git keeps them), and rework cards that add or change a classifier, verb or guard go through Plan-Review.
This is what made refusal-audit circle: four rework rounds grew its plan to 64 cards, of which 54 were done, which overflowed Master's launch prompt, let Master claim done with ten cards unrecorded, and exhausted the bounce budget.

## Also folded in

- `review-attack-surface`: the discussion and plan rubrics ask what a new verb or downgraded guard can skip or let through.
- `burler-focus-prior-round`: three refusal-audit friction notes hit the same ordering trap.
- refusal-audit self-report 5: Webster kept its earlier card list after rework; "Still to run" (`3613d5076`) patches the symptom, generations remove the cause.

## From `rework-design-review`: PR-Rework cards that change a classifier, verb or guard get a Plan-Review pass

loom-recipe.yaml:37 exempts PR-Rework from Plan-Review. Keep the exemption for text-only findings; route cards that create or change a classifier, verb or guard through Plan-Review.

### Rework should not drag the implemented plan along (operator, 2026-10-01)

PR-Rework appends its cards to the same plan, and Webster resumes the same run, so Master's prompt and context carry every earlier card's index and progress line.
In refusal-audit round 4 that was 64 batches with 54 done; the rendered prompt outgrew the launch limit (fixed on main in `e39909bbc` by handing Master the prompt as a file, which treats the symptom only).
Done cards are dead weight: Master never re-runs them, but they fill its context and every fork inherits it.
Want: each rework round is its own plan generation; Webster sees only that round's cards plus `00-overview.md`, and the earlier cards are archived (git keeps them), so prompt and context stay flat however many rounds a task takes.


## From `review-attack-surface`: Reviews ask what a new verb or downgraded guard can skip or let through

From a Fable process analysis of refusal-audit, 2026-10-01: Discussion-Burler round 1 F5 itself wrote "goto places no restriction on targets" (reviews/discussion/round-1-review.md:142); Plan-Review checks fidelity to the decision record and so cannot question it; Webster-Review reviewed the diff as code.
Want: loom-rubric-discussion-review.md (and the plan rubric) asks, for every new verb or downgraded guard, what it can now skip or let through; pairs with orch-discussion-review.

## From `burler-focus-prior-round`: Burler: round-N focus directive conflicts with the read-prior-rounds-last rule

Friction from run `operator-surface`, Burler Webster-Review round 2 (stencils from `d619100af`).
The round-2 focus directive asked the reviewer to check the round-1 F4 site list and the round-1 fix commits, which requires reading `round-1-review.md` and `round-1-fixer-report.md`.
The round prompt's instruction 2 says prior-round files are read only after the review is saved, so the focus file handed over in instruction 1 cannot be followed without breaking it; the reviewer read them early and said so.
Fix direction: the focus directive restates the sites it depends on, or the prior-round rule exempts what a focus directive names.