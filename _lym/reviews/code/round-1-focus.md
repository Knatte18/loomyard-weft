---
round: 1
focus:
  - "contracts/stencils/burler/burler-review-format.md: trace which templates and steps render this format (solo, cluster fan, assisted, consolidation, parent consolidation, fix-check). Does every path that writes or rewrites a `fix` carry the new fact checks, as the change description claims?"
  - "burler-review-format.md, new `decision` route: compare it with the existing meaning of class `decision` and with how the fixer step, the judge and any other stencil handle such a finding. Can a fixer and the run act on a `fix` that offers two choices without contradicting another rule?"
  - "burler-review-format.md, fact list: read it against the amended entry's cases from issues #285 and #288 (kit, recording, stand-in, tier able to run it; package clause and export status; callee chain). Is each asked check carried, including what the `fix` says when a fact is unverified or only partly readable?"
  - "contracts/stencils/burler/burler-step-fix.md: does the new unverified-fact instruction agree with the three dispositions, their order, and the handling of `site not found` and `decision` findings in the same file?"
  - "Issue #291 cases: set each of the five against the claim and cross-card wording in loom-block-plan-cards.md and loom-rubric-plan-review.md. Is each case caught by that wording, and do writer and reviewer carry the same obligations?"
  - "First-caller rule in loom-plan-spec.md, loom-block-plan-cards.md and loom-rubric-plan-review.md: do the three agree with each other and with the granularity, helper, public-surface, dependency-order and `Create` rules around them? Test it against a function with several callers, with no production caller in the plan, a method, and a caller that is itself not yet reachable from a `main` package."
  - "Other places that state card granularity or plan-writing and plan-fixing rules (the other stencils under contracts/stencils/, the specs, docs and PATTERN files): does any of them still say what the diff now contradicts, or fail to receive the new rule where the writer or fixer of a plan needs it?"
  - "_lym/landing/summary.md: check each claim against the diff and the stencil rendering paths, and check its `## Merge danger` door and blast radius against what the changed stencils can affect."
---
The diff is prose only, in stencils and the plan spec, so the review rests on reading that text as the diff leaves it and on its neighbouring rules.
The decision record's added entry rules out the red-first check and tests of stencil prose.
