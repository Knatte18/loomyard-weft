---
round: 1
focus:
  - "Cards 10 to 13: does every planglyph pass that reads an `ambiguous` answer classify it through `classifyAmbiguity` or the `partitionedAnswers` map, per the shared decision `classifier-takes-the-worktree-root`, with no pass still treating a partitioned member as ambiguous?"
  - "Card 1: after the waiting-turn-end rule and its dead fields are dropped, does any reader, test or doc still reference them, and does a gated run with an outstanding shell still never count the turn end at once?"
  - "Cards 5 to 7: are all recovery sessions recorded on the batch and counted as the run's own writes, and does a terminal stuck or dead recovery exit non-zero with the digest fields?"
  - "Cards 2, 6 and 7: per the shared decision `wording-cards-grep-the-old-text`, does any test or doc still assert the old message text?"
  - "Per-card mechanical check: for each Rename and Delete group in the cards, did the type-specific check run against every target in the group, not only the first?"
  - "Test economy: do the new tests in cards 9 to 13 duplicate coverage already given by the classifier's or the public passes' tests, and is every added behavior covered?"
---

Seed focus from the plan's card index and shared decisions.
Cards 3, 4 and 8 carry no entry beyond ordinary diff review.
