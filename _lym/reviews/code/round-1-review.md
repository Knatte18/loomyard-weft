---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-rubric-code-review.md:109
    summary: The code rubric's Test economy bullet still limits the prose-test finding to a stencil, while the plan rubric and the new Wording criteria bullet cover spec, doc and other text
    fix: In contracts/stencils/loom/loom-rubric-code-review.md, change the Test economy sub-bullet "a new assertion on a stencil's static prose, naming the assertion;" to "a new assertion on the static prose of a stencil, spec, doc or other text, naming the assertion;", and change the exemption at the end of the next sub-bullet from "a stencil's static prose is exempt" to "the static prose of a stencil, spec, doc or other text is exempt".
    origin: lens:code-affected-set
---

### [LOW] Code rubric's Test economy bullet is narrower than the rule the diff adds beside it

**Location:** contracts/stencils/loom/loom-rubric-code-review.md:109
**Issue:** The diff widens the plan rubric's prose-test finding from "a stencil" to "a stencil, spec, doc or other text" (`loom-rubric-plan-review.md:59`) and adds a Wording criteria bullet to the code rubric saying a test asserting the prose never stands in for the review check (`loom-rubric-code-review.md:103`).
The code rubric's own Test economy sub-bullets (lines 109 and 110) still say "a stencil's static prose", so a new test asserting a spec's or doc's prose is not named as a finding there, and a behavior with no test is exempt only when it is a stencil's prose.
The two rubrics now disagree on scope, though the decision record asks both to apply the same rule.

## Handler's holistic check against the decision record

- Expected bullet 1 (plan rubric): lands at `loom-rubric-plan-review.md:59-65`. A wording criterion is met by review, a card plans no test for it, a card that does is a finding whose fix cuts the test, and a missing test is never a finding.
  The one deleted line is the old "assertion on a stencil's static prose" clause, rewritten in a widened form; no other rule is dropped or weakened.
- Expected bullet 2 (Discussion prompt): lands at `loom-template-discussion.md:88-89`. It phrases criteria as testable behavior, marks a wording criterion as a review check, and points at the spec's rule without restating the mark.
- Expected bullet 3 (verify an existing-coverage claim in the code before cutting a test): lands at `loom-rubric-plan-review.md:64-65`. No file is named for it in the entry; the plan rubric is the plan review's prompt content (its header comment says it is interpolated into the review prompt), so placing it there is sound and the generic burler prompt stays shared.
- Expected bullet 4 (Discussion spec): lands at `loom-discussion-spec.md:88-90` beside the other acceptance-criteria rules, and agrees with the stencil and both rubrics on the `Review:` mark.
  `CheckDecisionIDs` (`internal/discussionparser/decisionids.go:70-79`) matches a decision id as a whole word anywhere in the bullet, so the prefix needs no parser change.
- Expected bullet 5 (code review rubric): lands at `loom-rubric-code-review.md:100-103` and conflicts with no existing test-economy bullet apart from the narrower wording in F1.
- The summary and the commit message match the diff. No `Pattern-Departure:` trailer is present and no PATTERN change is declared. No Go file changed, so no test is owed.

## Rejected

- Lens code-affected-set F-B (rule documents `loom-block-plan-format.md:16`, `loom-plan-spec.md:27` and `pattern/PATTERN-test-speed.md:155` still say "stencil's static prose"): the entry asks for five changes in four named places and none of these files; widening the plan writer's format block, the plan spec and a PATTERN entry is scope the answer key does not license, and the plan rubric's own rule already cuts such a test at review.
