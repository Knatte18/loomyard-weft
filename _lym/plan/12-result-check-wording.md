# Card 12 — result-check-wording

**Prosa:**
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils/landing/landing-template-pr-review.md`
- `contracts/stencils/landing/landing-template-describe.md`
- `contracts/stencils/darn/darn-template-write.md`

**Intent:** The wording criteria of D1, D4 and D7, checked by Code-Review reading the text, so no test is planned.

- `loom-rubric-code-review.md`, `## Also flag` (D1), three bullets worded for any language and hub:
  **Loosened checks**: an assertion, tolerance or limit the diff widens or removes in a test that exists at the review base is a finding unless a `Loosened-Check:` trailer on any commit of the review range names it and a measured cause; a cause that is no measurement, such as "flaky" or "to make it pass", is a finding too.
  **Tests still check what they say**: a test whose limits changed still checks the behaviour its name and doc comment state, and one whose new limits admit the failure its name rules out is a finding.
  **Claims of agreement or improvement**: a claim in the change description or a commit message of the range that a result agrees with a reference or improves carries before and after figures from the current code; a claim without them, a figure the change itself made obsolete, and an agreement claim across a set-up difference it does not name are each a finding.
- `landing-template-pr-review.md` (D4): a section before the parent's review tells the agent to run `lym loom pr-findings <file>` on every set of findings it receives before fixing any of them, the parent's review and a list of findings the operator gives, writing them first in burler's review format with its verdict line; on a send-back it ends its turn, and on fix-here it fixes them as today.
  A single change the operator asks for is no findings set and goes straight to a fix.
  The parent-review section's "Fix the findings" step runs the verb first.
- `landing-template-describe.md` and `darn-template-write.md`'s Step 5 (D7): a claim of agreement with a reference or of an improvement carries before and after figures from the current code, taken from the commits or the drift verdict, and a claim with none is left out.

**Commit:** `12: result-check-wording`
