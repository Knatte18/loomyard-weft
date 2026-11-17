---
round: 2
focus:
  - "Seat catalog completeness: the burler review table gains its assistant seat and an assisted chair stencil after the table literal is built, in `internal/burlerengine/assist.go` (`withAssistant`, `assistSeat`) and called from `review.go`. Check whether cards 6 and 7 let the seat comparison and the literal scan reach that seat, since the decision says every seat in `seatengine`'s tables is compared."
  - "Test-only reachability under `PATTERN-test-only-code`: for each new exported or package-level function in cards 2, 6 and 8 (`shuttleparity.Seats`, `shuttleengine.Manifest.Omitted`, `manifestEngine`, `seatsOf`), name the production call chain from a `main` package that reaches it. A function only tests call is the finding."
  - "Parity fixture skills: Claude's manifest lists one skill-load item per skill named by the load message, and Codex lists one only for a skill whose SKILL.md sits under a skill root. Check that the fixture in card 6, reused by card 7, holds every skill the catalog's seats and the producer table's specs name."
  - "Unmet-requirement skip: the plan skips specs whose requirements Codex's `Start` would refuse, though `Prepare` does not refuse them. Check whether the decision record carries that exemption, or whether the skip narrows the every-seat and every-producer comparison without one."
  - "Card 6 test coverage: check that each refusal or error branch card 6 adds (`PrepareEach` on a failing `Prepare` or an absent record, `seatsOf` on a failing builder) is reached by a named test row in card 6 itself, not only by a later card."
  - "Test economy in card 1: check each planned row of the new standing-passage test against the existing `TestPrepare_AppendsStandingPassages` in `internal/shuttleengine/claudeengine/prepare_test.go`, and read that test before judging overlap."
  - "Copy and warning wording in card 2: compare the described behavior of `Runner.launch` around `saveRunState` and `awaitStartup` in `internal/shuttleengine/run.go` against the card's statement about what a retried start attempt leaves, and check that each `copyManifest` warning branch names a way forward that fits its own cause."
---

The focus lists areas to settle, not verdicts.
