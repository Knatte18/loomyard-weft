# Wording criteria are checked by review, never by a prose test

The plan rubric's test-economy rule and a decision record's acceptance criteria pulled opposite ways when a criterion was phrased as a rendered-text check ("the rendered stencil names X").
The planner planned a test for it, the plan review asked to cut that test, and the dispute repeated every round ([issue #3](https://github.com/Knatte18/loomyard/issues/3)).
The operator settled it for test economy: a criterion about wording is checked by the code review reading the text, never by a test that asserts the prose.

## Changes

- `contracts/specs/loom-discussion-spec.md`: a new compaction rule says a criterion is behavior a test can check, or a review check.
  A criterion that a stencil, spec, doc or other prose names, says or contains a wording opens its bullet with `Review:`; the code review checks it by reading the text, and no test asserts that prose.
- `contracts/stencils/loom/loom-template-discussion.md`: the interview's Testing item tells the writer to phrase each criterion as testable behavior and to mark a wording criterion as a review check, pointing at the spec's rule rather than restating it.
- `contracts/stencils/loom/loom-rubric-plan-review.md`: the Test economy bullet widens the prose-test finding from stencils to any spec, doc or other text.
  A wording criterion, marked or not, is met by review: a card plans no test for it, a card that does is a finding whose fix cuts the test, and the criterion's missing test is never a finding.
  Before relying on a claim that an existing scan or test already covers a wording to cut a test, the reviewer reads that scan or test in the code and confirms it; an unconfirmed claim grounds no such finding.
- `contracts/stencils/loom/loom-rubric-code-review.md`: a new Wording criteria bullet checks each wording criterion by reading the text the diff leaves.
  With a plan the criteria are the record's `## Acceptance criteria`; with a decision record alone they are what the amended board entry expects.
  A wording the criterion asks for that the text lacks is a finding, and a prose test never stands in for the check.

## Design decisions

- The mark is a `Review:` prefix on the criterion's bullet. The decision-id check still finds `D<n>` anywhere in the bullet, so the prefix needs no parser change.
- Both rubrics apply the rule to an unmarked wording criterion too, so records written before this change and darn records, which carry no criteria of their own, get the same treatment.
- The plan review's verify-before-cutting rule lives in the plan rubric, which is the plan review's prompt content; the generic burler reviewer prompt is shared by every perch and stays unchanged.

No Go code changes, so no test is added; the change is wording the code review checks by reading it.
