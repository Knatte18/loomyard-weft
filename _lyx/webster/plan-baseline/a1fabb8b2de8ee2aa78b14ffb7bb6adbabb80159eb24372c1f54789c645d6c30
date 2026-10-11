# Card 2 — reviewer-answers-dispute

**Edit:**
- `internal/burlerengine#priorRoundsBlock`
- `internal/burlerengine#TestComposePrompt_PriorRounds`
- `internal/burlerengine/doc.go`

**Uses:**
- `contracts/stencils/burler/burler-step-3-fix.md`

**Intent:**
Extend the closing sentence `priorRoundsBlock` writes for a hydrated round with D4's rule: a reviewer that re-raises a finding a prior fixer-report disputed states, in that finding's `**Issue:**`, why the dispute's evidence does not hold.
The phrase `deferred and disputed items` stays, as does the clean-room order: the reviewer forms and saves its own findings first, reads the prior rounds' files only after, and amends the matching finding's `**Issue:**` in its saved review before it ends its turn.
The first-round fallback sentence is unchanged, and the function's signature does not change.

In `TestComposePrompt_PriorRounds`, the `prior round` subtest gains an assertion on a distinctive phrase of the new rule (for example `why the dispute's evidence does not hold`) beside the existing `deferred and disputed items` assertion; no new top-level test, per `PATTERN-test-economy`.

In `internal/burlerengine/doc.go`, the paragraph on the two exceptions to fix-everything states the same rule as the stencil Card 1 writes: a dispute on either ground (`premise false`, or every addressing change makes the target worse) with cited evidence in the Disputed section, the fix-with-own-change outcome marked in the fixer-report's changes list, the unchanged deferred meaning, and that severity, size, cost or rubric disagreement never justify a dispute and a dispute never converges a segment on its own.
Add one sentence that a reviewer re-raising a disputed finding answers the dispute's evidence in that finding's Issue.
Every edited comment line follows `PATTERN-comment-line-breaks`, which the card gate's `lyx loom lint-comments` checks.

**ImpactSummary:** One prompt-string function and its one test assertion in burlerengine plus the package doc; the stencil files and the rendered prompt's other blocks are untouched.

**Commit:** 2: Reviewer answers a prior dispute's evidence when re-raising
