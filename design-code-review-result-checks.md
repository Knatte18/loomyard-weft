# Code-Review lets result-level issues through to PR-Review

`code-review-result-checks` · Note · enhancement, burler, stencil, drift

Imported from [issue #139](https://github.com/Knatte18/loomyard/issues/139).

## Seen

A darn run (profile solid) passed Code-Review and reached PR-Review with substantive findings left: 17 items, of which the main ones were about results, not code.

- Test limits widened 10-30x without a named cause, so a golden test no longer checked the behaviour its name states (steady turning).
- Model departures kept on the strength of measurements the change itself had made obsolete.
- A set-up difference between the model and its reference (one side had a fluid torque the other lacked) that the description presented as agreement.
- No before and after figures for the comparison the task existed to move.

The PR-Review agent then has to make a full measuring and fixing round, which PR-Review is not meant for: at PR time only small things should remain.

## Why Code-Review missed it

The code reviewer reads the diff against the task and checks code quality and correctness.
It is not told to check that assertions, tolerances and measured claims in the change still hold together, and a widened limit in a test looks like an ordinary edit.

## Idea

1. Add result checks to the code reviewer's brief in every recipe, generic across hubs:
   - a widened assertion, tolerance or limit names its measured cause, or is a finding;
   - a test whose limits changed still checks what its name and doc state;
   - a claim of agreement with a reference or of an improvement carries before and after figures in the change description.
   Domain rules, such as "a kept departure from a reference model is measured on the current code", belong in the repository's PATTERN, which reviewers already get.
2. Drift compares results before and after, but a test whose limits are widened in the same change passes on both sides, so drift sees nothing.
   Let Drift also flag every assertion, tolerance or limit the change widens or removes in an existing test, and require a stated measured cause for each, as it does for a moved baseline value.
3. When PR-Review findings are more than small (by count or by a severity the reviewer gives), send the run back to Code-Review / Code-Burler instead of having the PR-Review agent fix them, and record that the earlier review missed them, so the miss is visible.
