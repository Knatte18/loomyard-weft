---
round: 1
focus:
  - "Cards 1 and 2 (tokencount): settle whether the turn-start and 30-minute stall-cap idle rules and the run lower bound match the plan's cards, including the report line for transcripts left out, and whether the package doc in `main.go` was updated."
  - "Cards 3, 4 and 5 (loom help and refusals): settle whether each changed refusal has its `contracts/specs/refusal-spec.md` row and a test that reaches it, and whether any recovery text names something outside `lyx loom start`, `lyx loom step`, `lyx shed seed` and a plain `rm <path>`."
  - "Card 7 (batten notices): settle whether the quiet, api-error and driver-dead notices lead with the child's state and row, and whether battenshed's `doc.go` matches."
  - "Test economy across all cards: settle whether any new top-level test beyond card 3's one help test was added, whether any package passes its row in `cmd/lyx/testdata/test-budget.yaml`, and whether each added behavior has coverage."
  - "Per-card mechanical checks: settle whether each card's group-type checks ran against every target of the group."
---

The plan is seven small independent fixes; the focus above is where the diff is most likely to diverge from the cards.
