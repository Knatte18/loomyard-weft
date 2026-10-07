---
round: 3
focus:
  - "In `_lyx/plan/11-tmux-tier-green.md`, does the card's premise match what the repository and the decision record support, and does it tell the implementer what to do when `go test -tags tmux ./...` already passes on a clean tree?"
  - "In `_lyx/plan/11-tmux-tier-green.md`, is the cause site left to the captured failing output rather than asserted, and does the card's scope (test file only versus reed production code) match where a failure could arise?"
  - "In `_lyx/plan/09-mark-guards.md`, is every file in the `**Edit:**` list one the card's own marking criterion marks at least one test in, or does the card state that the list is a candidate set?"
---

Card 12 depends on card 11 leaving the `tmux` tier passing, so check the two cards read consistently at that seam.
