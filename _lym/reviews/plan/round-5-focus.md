---
round: 5
focus:
  - "Card 22 (`_lym/plan/22-driftcli-retire-verbs.md`): settle whether any `Delete` target of the card is still referenced by production code in `internal/driftcli` or `internal/driftengine` after every card has landed. Start with `internal/driftcli#compareLockRetry`, which `worktree.checkDeps` in `internal/driftcli/wiring.go` reads for the check verb's `Compare` seam under `diff: command`, and check each other deleted symbol the same way against the callers cards 14 and 26 keep."
  - "Card 25 (`_lym/plan/25-driftengine-retire-acceptance.md`): settle whether the card's deletion of `internal/driftengine#testFiles` leaves a live reference, since `internal/driftengine/check.go` references `testFiles` in `Check` around line 274, and whether an earlier card moves that reference before card 25 deletes it."
  - "Cards 20 to 26, each `Delete` target: settle for every deleted symbol whether a card at the same position or later still reads it, using the landed code of cards 1 to 19 as the starting set of readers."
---

The plan gate blocks a `Delete` of a symbol a surviving caller still references, so the questions above are about what the gate cannot know: which callers the earlier cards keep.
