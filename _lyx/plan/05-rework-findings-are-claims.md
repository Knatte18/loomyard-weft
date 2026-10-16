# Card 5 — rework-findings-are-claims

**Prosa:**
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** A rejection's findings are a reviewer's claims, never an operator ruling (D1), and the rework writer never asks the operator (D4).

`loom-template-rework.md`:

- The role paragraph says the producer reads the findings of the reviewer that rejected the pull request, the alignment agent or the parent's review, and weighs them as a review against the code and the decision record, never as an operator ruling.
- "You never interview and never ask." becomes: it never interviews and never asks the operator, and asks its parent as the parent directive says.
- Step 2 item 1 calls the record's `findings` field the rejecting reviewer's findings, not the operator's.
- Step 3's coverage bullet gains the route, pointing at the plan stencil's late scope-addition route in its Step 1 (already read in Step 1 here) without restating it (`PATTERN-producer-pointer-rule`):
  a finding that would overturn a Decision of the record is one case of that route, planned only once the writer has asked its parent to record it with `lyx loom decision add` and the record carries it;
  if the parent declines, or no parent is recorded, the writer stops the whole rework: it plans no further finding, writes no `{{.coverage_path}}`, and ends its session reporting the finding, so the missing output halts the run for the parent;
  a finding that overturns no Decision is not a late scope addition under that route and is planned directly, licensing its card's scope, as this prompt's own exception to the plan stencil, which its Step 1 allows;
  every other finding, one adding scope the record does not cover included, is covered by a card with no recording.
- Step 5's sentence that `{{.coverage_path}}` is written last stays; the stop above is the one case where it is never written.

`loom-rubric-plan-review.md`'s `Fidelity to the answer key` bullet, in its rework sentences: a finding in `findings.md` is a reviewer's claim the rework plan answers, never a ruling; it licenses a card's scope unless it overturns a Decision of the decision record; a card carrying a finding that overturns a Decision with no `Added after Discussion` entry recording it is a finding.
The rest of the bullet (the live-generation round rule, the `prior-generation/` exclusion) stays.

The sentences name no denied command (`PATTERN-no-denied-recovery`), and the ask sentences carry the `never` bound the `TestStencils_NoUnboundedAskOperator` scan reads; both scans run in the card gate's guard step.
The changes are static prose, which no test asserts.
