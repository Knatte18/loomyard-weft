---
verdict: CONTINUE
rationale: "Round 2 carries a BLOCKING design finding, F8: a test added by fix commit ce6dee700 does not open with tiergate.Require, which PATTERN-test-speed requires."
---

F8 cites `PATTERN-test-speed`, a listed constraint, so it stays BLOCKING.
The review notes that the rubric's `Do not flag` list leaves a failing test to the verify gate; the constraint entry wins over that rule.

F9 carries F7 as a LOW `decision` finding that waits on the parent's ruling.
No finding was relabelled, and the review has no `## Focus departures` section.
