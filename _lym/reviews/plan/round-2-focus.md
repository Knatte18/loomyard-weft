---
round: 2
focus:
  - "Launch wiring: for every production site that builds shuttle engines (`shuttleengines.New` or `NewForWorktree`), check that some card switches it to the worktree-aware constructor, including launches made through `battencli.ShuttleRunner`."
  - "Usage limit on the skill-load turn: settle whether a card handles a limit hit during the load turn that `launch` runs before `Wait`, so `blocked_until` is recorded and the switch is asked, and whether its planned test drives that path through `Start`."
  - "Refused resume on the start path: settle whether the card's decision site sees the first API-error turn end, given that the load hold consumes API-error turn ends before `loadSkillsThenPrompt` regains control."
  - "Fresh-reviewer fixes: for each card that gained text in this round, check that its Intent, targets, glyph arrows and planned tests agree, that no doc listed under `Extends` has sentences the card falsifies, and that no new test duplicates a public-surface test."
  - "Registry and shared-builder tests: for each new config key, seam func or record field, check that existing tests which pin the full key set or build the shared fixture are listed on a card."
---

Round 1 was not converged, so round 2 re-reviews the whole plan with these questions added.
