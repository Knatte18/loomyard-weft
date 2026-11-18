---
round: 2
focus:
  - "Card 7 (`_lym/plan/07-bouncer-review-inputs.md`): settle whether Code-Burler-Review's reviewers can read the review-input note of a generation the Bouncer starts in the same `lym loom run` process, given that `burlerReviewEntry` in `internal/shedrecipe/entries_burler.go` builds its rubric notes once when the shed is built, and that the plan names a test reaching the review row's rubric."
  - "Cards 8 and 12 (`_lym/plan/08-loom-pr-findings-verb.md`, `_lym/plan/12-result-check-wording.md`): settle whether the plan states where the PR-Review agent writes the findings file, given that `verifytree.DirtyPaths` in `internal/verifytree/verify.go` counts untracked files and that the verb and card 9's send-back refuse on a dirty tree; check that a test row reaches the chosen location."
  - "Cards 3 and 10 (`_lym/plan/03-drift-verify-loosened-checks.md`, `_lym/plan/10-recipes-pr-review-reroute.md`): for each seam or routing value the card changes, settle whether every existing test and test kit that builds or pins it (the shedrecipe drift test environment, the loomrecipe routing test) is listed under **Edit:** with the change stated in the Intent."
  - "Cards 7, 9 and 10: settle whether each doc comment, package doc and spec passage that the card's change falsifies, such as the Bouncer passage of the shed-recipe spec, the `PRReview` type doc and the claim that PR-Review's Stuck is always a Halt, is listed under **Edit:** or reworded by the card."
---

Round 1 did not converge: two BLOCKING scope findings and two MEDIUM design findings were open.
The entries above point at the areas where the plan changed or where its fix text is new.
