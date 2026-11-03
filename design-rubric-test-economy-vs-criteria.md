# Plan rubric's test-economy rule conflicts with decision-record acceptance criteria phrased as rendered-output checks

`rubric-test-economy-vs-criteria` · Task · enhancement, burler, loom, stencil · running · Code-Burler-Fix

The plan rubric's test-economy rule and a decision record's acceptance criteria pull opposite ways when a criterion is phrased as a rendered-text check ("the rendered stencil names X"): the planner plans a test for it, the plan review asks to cut that test, and the dispute repeats every round ([issue #3](https://github.com/Knatte18/loomyard/issues/3), seen in task `interactive-recipe`).

## Decision (operator, 2026-11-03)

Test economy wins: lym keeps its test count down and cuts tests rather than adds them.
A criterion that some text names, says or contains a wording is checked by the code review reading the text, never by a test that asserts the prose.

## Expected

- The plan rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`) says a criterion about wording in a stencil, spec, doc or other prose is met by review, so a card plans no test for it and a test asserting static prose is cut.
- The Discussion writer's prompt (`contracts/stencils/loom/loom-template-discussion.md`) phrases acceptance criteria as behavior a test can check, and marks a wording criterion as a review check, so the record no longer asks for prose tests.
- The plan review's prompt has it verify, against the code, any claim that an existing scan or test already covers a wording before it relies on that claim to cut a test.
- `contracts/specs/loom-discussion-spec.md` states the same rule where it defines acceptance criteria.
- The code review rubric (`contracts/stencils/loom/loom-rubric-code-review.md`) reads each wording criterion in the record and checks it in the diff.
