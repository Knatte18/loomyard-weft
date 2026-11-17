---
round: 4
focus:
  - "Card 6 creates package `internal/shuttleparity` with tests, and cards 7 and 8 add more: does the plan create a `TestMain` for it, as `internal/seatengine/testmain_test.go` does, so the `tmuxkit.Main` isolation guard in `cmd/lym/tmuxisolation_test.go` passes after card 6?"
  - "Cards 5 and 6 add production functions (`shuttleengines.NewWithSeams`, `seatengine.Specs`, `landingshed.FixReviewSeatTable`, `burlerengine.Engine.SeatTables`) that no main package reaches until card 8: for each prefix of the card order, does `cmd/lym/testonlycode_linux_test.go`'s `testOnlyCodeAllowed` admit them, and does card 8 remove the interim entries?"
  - "Card 6 `Seat.Specs` builds a spec per engine from a seat's `ModelChoice`: does the plan state which of Model, Effort and Version the spec keeps, so that `codexengine.validateChoice` and `claudeengine.Prepare` accept a seat whose model-spec carries an effort or version pin?"
  - "Cards 5 to 8 and the Edit glyphs: does each Edit card whose Intent changes a member's parameters or results (for example `internal/shuttleengines#build`) carry the re-sign arrow?"
---

The prior round's closing review found the issues above; the round-4 plan is a fresh read, so settle each question against the plan as it stands.
