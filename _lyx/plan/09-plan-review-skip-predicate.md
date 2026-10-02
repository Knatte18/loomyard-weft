# Card 9 — plan-review-skip-predicate

**Create:**
- `internal/loomshed/generation.go`
- `plan:internal/loomshed#PlanReviewSkippable` -> `func PlanReviewSkippable(deps PRReworkDeps) (bool, error)`
- `internal/loomshed/generation_test.go`

**Edit:**
- `internal/loomshed/prrework.go`

**Uses:**
- `plan:internal/planparser#ReviewExempt`

**Intent:**
Add the Go half of the Plan-Bouncer skip seam (D7): decide from committed state whether the live plan generation is exempt from Plan-Review.

The live generation's round is the highest rework round whose committed `record.json` carries a `class` and whose `first_card` equals the live overview's `first_card`; with none, the live plan is generation 0 (the overview's `generation-round-rule`, which this predicate is the authoritative reading of).
`PlanReviewSkippable` parses the live plan as committed at HEAD (`ParseCommittedPlan`) and reports true only when the live generation's round exists, its committed `record.json` records the class `exempt`, and `ReviewExempt` over those live committed cards also says exempt.
It returns an error when the committed plan or a committed record cannot be read or decoded, which the Bouncer turns into a real review.

Bound, stated in the doc comment: the inputs are not Go-only (the rework session writes the overview's `first_card`, and status commits sweep `_lyx/loom/rework/`, so an agent-written `record.json` can reach HEAD); re-classifying the live cards means a stale or forged record can at most skip review of a generation whose live cards are themselves exempt.
Generation 0, a plan rewritten by an operator `goto Plan-Write` (its `first_card` matches no round), a classless record and a `required` record never skip.

The round listing and committed-record reading `prRework` already does move into package functions in this file that both share, so the round vocabulary has one reader.

Tests (fake committed-file reader over temp directories): an exempt record over exempt live cards with matching `first_card` skips; a `required` record does not; generation 0 (no round) does not; a round whose `first_card` differs does not; an `exempt` record over live cards that classify `required` does not; when two rounds exist, only the highest match decides; an undecodable committed record returns an error.

**ImpactSummary:** additive predicate plus a move of two private round helpers out of `prrework.go`; PR-Rework's behaviour is unchanged.
