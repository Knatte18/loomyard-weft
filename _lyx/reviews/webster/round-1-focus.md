---
round: 1
focus:
  - "Cards 7 and 8, `internal/burlerengine/engine.go` and the `join` path: settle whether `Run` and `Resume` both end through `join`, and whether every failure rule (fixer done before the marker, `ErrHalfNotStopped`, review file changed after parse) is enforced there and nowhere else."
  - "Marker writing: settle whether lyx code alone writes the marker, only after the current attempt's reviewer is done, the fork audit passed (cluster rounds) and `ParseReview` succeeded, and whether `Run` removes a stale marker before either half starts."
  - "Stopping a half: settle whether every stop goes through `StrandRemover.RemoveStrandIfLive` on the strand guid, and whether a failed stop is never retried or archived over."
  - "Told geometry: settle whether `burlerengine` derives no marker path itself and receives it only on `Profile.ReadyMarkerPath` from `burlermarker.Path`."
  - "Card 3 and card 7 callers: settle whether the removed recipe `model`/`effort` keys and the changed `Shuttle` signature (`StartGated`, `Handle`) leave no stale caller, fixture, stencil reference or doc in `shedadapters`, `shedrecipe`, `burlercli`, `loomcli` and `shedfake`."
  - "Card 8 `ProbeRound`/`Resume` and the producer's two-strand probe and retry: settle whether each resume state (reviewer only, fixer only, both, neither live) reaches the right action, and whether a retry cannot start a second reviewer over a live one."
  - "Per-card mechanical check: for the Delete group of card 7 (the old round-orchestrator stencil) and the removed recipe keys of card 3, settle whether the type-specific check ran against every target in the group."
  - "Test economy: settle whether the new `burlerengine` and `shedadapters` tests cover the new behavior at the public surface, and whether any test duplicates another or tests a helper the public-surface test already covers."
  - "Docs and guards in the same commit: settle whether `doc.go`, `docs/overview.md`, `pattern/PATTERN-review-round.md`, `cmd/lyx/pathrules_test.go` and `cmd/lyx/testdata/test-budget.yaml` match the code each card changed."
---

Focus is set from the plan's Card Index and Shared Decisions; the committed diff decides which entries produce findings.
