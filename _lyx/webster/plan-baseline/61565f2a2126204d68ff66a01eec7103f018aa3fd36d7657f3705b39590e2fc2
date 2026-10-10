# Card 2 — pr-gate-budget-prose

**Prosa:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `docs/overview.md`

**Intent:** D2 of the decision record: the PR-Gate budget prose states the episode rule the engine applies, so no agent designs against a five-round-per-task bound the engine does not have (#526).

In the loom recipe's header comment, replace the two sentences "The gate's episode Stuck count never resets between rounds, so the budget is five rejection rounds per task." with the rule: a Done by any row of segment `PR-Review` ends the gate's episode, so `PR-Rework`'s Done resets the count after every rework; `max_bounces` bounds consecutive gate Stucks with no rework Done between them, which this recipe never produces, since every gate Stuck routes to `PR-Rework`.
The sentence that follows, on a rework round starting a new plan generation, stays.
In the darn recipe's header, reword the sentence on the edge's bound the same way: `max_bounces` is the value the status projection and the reject refusal report; a Darn Done ends the gate's episode and a Darn Stuck halts the run, so consecutive rejections never accumulate and the budget is never reached under this recipe.
Both blocks keep their fixed-column wrapping, the style of the comment block they sit in; `max_bounces: 5` stays on both PR-Gate rows (A2).
In the overview's loom section, the sentence on `reject <review-file>` keeps its first clause and replaces "refuses once the `PR-Review` segment's five rejection rounds are spent" with: its refusal once the gate's bounce budget is spent exists but is unreachable under the recipe, whose `PR-Rework` Done ends the gate's episode after every rework.
Every link in the file stays as it is, so PATTERN-markdown-link-integrity holds.

Grep the repository for "never resets" and "rejection rounds" before finishing: the Go comments the first phrase hits all sit in doc comments card 1 rewrites, including `internal/shedengine`'s `Routing.Bounces` doc, whose clause on a Burler episode that never resets is false; `internal/loomengine`'s "rejection round" names the rework directory layout and stays.
No test asserts either phrase, and the reject verb's refusal message in `internal/loomcli/reject.go` is unchanged (AC6).
