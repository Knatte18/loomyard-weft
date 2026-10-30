---
round: 4
focus:
  - "Card 14 (`_lym/plan/14-pr-review-session.md`) and `internal/landingshed/seam_enforcement_test.go`: does every package the card's `ensureSession` imports, including any needed to fill the `landing-template-pr-review` stencil, appear in the card's stated additions to `landingshedAllowedImports`, so `TestToldGeometryInvariant_AllowlistOnly` passes at that card?"
  - "Cards 11 and 13 (`_lym/plan/11-pr-review-row-opens.md`, `_lym/plan/13-pr-review-row-waits.md`): on a re-entry with a recorded pull request, how many `Status` reads happen in the first minute of the injected clock, and does the plan state where the wait's poll clock starts and name a test row that pins that count?"
  - "Card 11's recipe paragraph and the header comments of `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml`: does every comment that says PR-Review never returns Stuck or always returns Done get a stated rewrite once the row can Halt?"
---

Targets the three keys still open after round 3.
