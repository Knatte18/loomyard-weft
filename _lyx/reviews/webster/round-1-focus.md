---
round: 1
focus:
  - "Card 2 (Delete group): check that `assert-no-callers` ran and passed for every deleted symbol, including the `notes` group, the `promote-note` verb and the stale `cliTreeFindings` entries, and that `Board.PromoteNote` is gone once card 3 lands."
  - "Cards 3 and 4: check that no `tier` or `type` field, helper or doc string survives in the model, render, listing, promote and migration code, and that old-shape `board.json` entries load and persist on the next write as the plan states."
  - "Shared decisions: check that `Vocabulary` is set on every store `Board` builds, that the path-only predicate matches `boardCriticalSection` and skips label validation, and that each refusal in cards 3, 5, 6, 8 and 9 names the way forward."
  - "Cards 7 to 9: check that intake import, fold and close behave as the cards specify, including that a fold closes the issue with a pointer and that a close states its reason."
  - "Comment conventions: check new and changed doc comments against the target repository's CLAUDE.md and PATTERN.md rules, with a rule written there outranking inferred style, and check that `doc.go`, `docs/overview.md` and the D13 label list changed in the same commit as the behavior (card 10)."
---

Focus is set from the plan's cards and shared decisions only; no earlier round exists.
