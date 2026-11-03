# Check wording criteria by review, never by a prose test

The plan review's test-economy rule and a decision record's acceptance criteria pulled opposite ways.
A criterion phrased as a rendered-text check ("the rendered stencil names X") made the planner plan a test, the plan review then asked to cut it, and the dispute repeated every round.
Test economy now wins: a criterion about wording in a stencil, spec, doc or other prose is checked by the code review reading the text, and no test asserts that prose.

Closes #3

## What changes

- `loom-discussion-spec.md`: the acceptance-criteria rules gain one bullet.
  A criterion is behavior a test can check, or a review check; a wording criterion opens its bullet with `Review:`.
- `loom-template-discussion.md`: the Testing prompt tells the Discussion writer to phrase each criterion as testable behavior and to mark a wording criterion as a review check.
- `loom-rubric-plan-review.md`: a card that plans a test of static prose in a stencil, spec, doc or other text is a finding whose fix cuts the test.
  A wording criterion with no planned test is never a finding.
  Before a finding relies on a claim that an existing scan or test already covers a wording, the reviewer reads that scan or test and confirms it checks that wording; an unconfirmed claim grounds no finding that cuts a test.
- `loom-rubric-code-review.md`: a new "Wording criteria" bullet has the reviewer read each wording criterion against the text the diff leaves.
  The test-economy bullets widen from a stencil's static prose to the static prose of a stencil, spec, doc or other text.

## Design decisions

- The rules apply to a wording criterion whether or not its bullet opens with `Review:`, so a record that omits the marker does not bring the dispute back.
- With a decision record alone and no plan, the wording criteria are what the amended board entry expects.

## To check by hand

Only stencil and spec prose changed, so no test was added or run.
Read the four edited files for consistency of the `Review:` marker rule across them.
