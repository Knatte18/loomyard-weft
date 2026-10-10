---
round: 2
focus:
  - "Card 1 (`_lyx/plan/01-gated-turn-end-holds.md`) removes `Run.startedFresh` and `gatedArrival` from `internal/shuttleengine/run.go`: settle whether every test file in `internal/shuttleengine` that reads either field, and every test whose expectations rested on the removed fresh-gated at-once exit, is on the card's `**Edit:**` list or named in its Intent."
  - "Card 4 (`_lyx/plan/04-verify-gate-way-forward.md`) has `NewVerifyGate` in `internal/websterengine/verifygate.go` call a helper that returns an error: settle whether the card decides how that error is handled, and whether every caller of a re-signed constructor is listed under `**Edit:**`."
  - "Cards 10, 11 and 12 let a partitioned ambiguous answer through a guard: settle whether each card's Intent states what bounds what now passes."
---

Focus targets the places where the round-1 plan changed shape; it caps no severity and does not replace a full review.
