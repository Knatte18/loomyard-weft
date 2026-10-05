---
round: 1
focus:
  - "contracts/stencils/claims_test.go: does each changed `wordingClaims` row match phrases actually present in the edited stencil or rubric, and does no card-owned file lose a pinned phrase without its claim moving?"
  - "contracts/specs/loom-plan-spec.md: do the edited card-criterion, granularity and worked-example prose stay consistent with each other, and do the worked-example card bodies and the planparser golden fixture stay byte-identical as Shared Decisions requires?"
  - "contracts/stencils/loom/loom-rubric-plan-review.md and loom-rubric-webster-review.md: does each redundant-test finding condition point at `PATTERN-test-economy` and restate it at most as one finding condition, per `PATTERN-producer-pointer-rule`?"
  - "contracts/stencils/burler/burler-step-3-fix.md and loom-template-discussion.md: does the added wording match card 4's intent (a fix adds a test only for a coverage gap; must-cover scenarios are behaviors) without contradicting neighbouring lines?"
  - "Per-card mechanical check: for each card, did the card's own group checks run and pass against every group's targets, not only the first label's?"
  - "Markdown changes in the diff: semantic line breaks (one sentence per line), and table cells and blockquotes on one line."
---

Seed focus from the plan's four cards and the diff stat.
