# Card 5 — lighter-convergence-rule

**Edit:**
- `internal/shedadapters#roundFacts`
- `internal/shedadapters#computeRoundFacts`
- `internal/shedadapters#renderRoundFacts`
- `internal/shedadapters/bouncerfacts_test.go`
- `internal/shedadapters#decisionRuleMarker`
- `internal/shedadapters/bouncerprompt_test.go`
- `internal/shedadapters/doc.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `contracts/stencils/claims_test.go`
- `PATTERN.md`
- `pattern/PATTERN-review-round.md`

**Uses:**
- `internal/shedadapters/bouncercarryover.go`

**Intent:**
Make a review segment converge from round 2 on once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round (D1), and amend `PATTERN-review-round` in the same commit (D7).

Round facts (`bouncerfacts.go`):

- `roundFacts` gains an earlier-open list, which `computeRoundFacts` fills: every key whose status is `open` in at least one ledger before the latest round, with the ledger rounds it was open in, sorted by key.
  No signature changes: `computeRoundFacts` already takes the run directory and the round, and `decisionRuleMarker` already takes the round.
  At round 2 it lists a key open in round 1 only, which the recurring-key list never does.
- Each earlier ledger (1..round-1) that is missing or fails `parseLedger` adds a parse-error line naming its round and the error, so a degraded read never empties the guard.
- `renderRoundFacts` renders it as its own section after the recurring keys, under a heading naming it as the keys open in an earlier round, with a fixed line when the list is empty; output stays deterministic.
- The recurring-key list and `circlingEvidence` are unchanged, so CIRCLING and its Go guard read exactly as today.

Decision rule (`decisionRuleMarker`, still the single holder of the rule's variants):

- Round 1 renders today's rule unchanged: `CONTINUE` while the latest round carries a gating-class finding at MEDIUM or worse, or any BLOCKING finding; otherwise `CONVERGED`.
- Round 2 and later render the lighter rule: `CONTINUE` while the latest round carries any BLOCKING finding, or a gating-class finding at MEDIUM or worse that the judge maps to a key in the facts' earlier-open list, unchanged or reopened; otherwise `CONVERGED`.
  It states that such a fresh-key finding is carried into the decision record's `## Open risks` for a later check, and that any parse-error line in the earlier-open list means the judge rules by round 1's rule.
- The CIRCLING bullet from the checkpoint on and the latest-round parse-error bullet are unchanged in both variants.

Judge stencil (`bouncer-template-judge.md`):

- Reading-order item 1 names the earlier-open list among the facts.
- "Fixed findings never count as convergence: a fix is new text no fresh reviewer has seen." is reworded to the new bound: a fix converges only for a key first raised in the closing round, from round 2 on, where the decision rule says so, and only because Go carries it into the decision record; any other fix is new text no fresh reviewer has seen.
- The paragraph on the recurring-key list covers both ledger-derived lists: a listed key counts as gating when the latest review's finding the judge maps to it is gating-class at MEDIUM or worse, or BLOCKING.
- The header comment's description of `{{.decision_rule}}` names the round-1 and round-2-and-later variants.
- No sentence contradicts D1; `claims_test.go` replaces the "Fixed findings never count as convergence" claim with a claim on the new bound and adds a `mustNot` for the old sentence.

PATTERN:

- `pattern/PATTERN-review-round.md`'s bullet "A review segment converges only on a judge verdict over a fresh review, and fixed findings never converge on their own." becomes: a segment still converges only on a judge verdict, and from round 2 on a fix of a key first raised in the closing round may converge, carried to the decision record's `## Open risks`; every other clause stays.
- `PATTERN.md`'s `PATTERN-review-round` one-liner is reworded to match.

Bound: a round-2-or-later round may now converge over fixes of fresh MEDIUM `design` keys no fresh reviewer has seen.
It never lets through a BLOCKING finding, a MEDIUM `design` finding the judge maps to a key in the earlier-open list, a MEDIUM `design` finding in a round whose earlier ledgers did not all read, or any round-1 fix; and every finding it closes over reaches the record through cards 2 and 3.

Tests:

- `bouncerprompt_test.go`: `decisionRuleMarker` renders today's rule at round 1 and the lighter rule from round 2, both with the CIRCLING bullet from the checkpoint on, and the round-2 rule names the earlier-open list and the parse-error fallback to round 1's rule; extend the existing table rather than add a test.
- `bouncerfacts_test.go`: the earlier-open list holds a key open in round 1 only at round 2, a key open in several earlier ledgers with its rounds, and a parse-error line for a missing and for an unparseable earlier ledger; the render carries the section and its empty-list line, and stays byte-identical across two writes.

Update `doc.go`'s checkpoint-judging paragraph for the round-dependent rule.

**ImpactSummary:** Changes the judge's prompt rule and facts file for every review segment from round 2 on; Go's verdict parsing, CIRCLING guard and settle logic are untouched.
