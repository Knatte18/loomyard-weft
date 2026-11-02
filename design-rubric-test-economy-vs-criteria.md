# Plan rubric's test-economy rule conflicts with decision-record acceptance criteria phrased as rendered-output checks

`rubric-test-economy-vs-criteria` · Note · enhancement, burler, loom, stencil

Imported from [issue #3](https://github.com/Knatte18/loomyard/issues/3).

## What happened

During a plan review round, a finding asked to cut a test's static-prose assertions under the rubric's test-economy rule.
`_lym/plan/lens-review.md` already recorded the Plan-Write lens pass disputing the same cut, because the decision record's acceptance criteria call for those checks ("the rendered stencil names X").
The review's claim that the stencil-wide scans cover the wording was false for all but one item; two self-check lenses caught it.
The fixing agent resolved it by stating in the card that the criteria are met by review of the stencil text.

## Why it is abnormal

The rubric and the decision-record acceptance criteria pull in opposite directions.
A criterion phrased as "the rendered stencil names X" can be planned either as a test or as a review check, and nothing in lym says which.
The same dispute is re-litigated each review round, and the review's coverage claim can be wrong without any check noticing.

## What to do

Settle once, in the plan rubric or the decision-record template, whether a criterion phrased as "the rendered text names X" is planned as a test or as a review check.
Have the review verify claims that stencil-wide scans cover a given wording before it relies on them.

Provenance: task `interactive-recipe`; note `burler-plan-r1.md`.



_Moved from #908 in the repository before it was recreated._
