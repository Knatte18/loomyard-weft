# Card 10 — Delete the strand removers shuttle's stop verb replaced

**Delete:**
- `internal/burlerengine/remover.go`
- `internal/burlerengine/remover_test.go`
- `internal/testkit/shedfake#StrandRemover`
- `internal/testkit/shedfake#StrandRemover.RemoveStrandIfLive`
- `internal/websterengine#removeStrandIfLive`

**Edit:**
- `internal/testkit/shedfake/shedfake_test.go`
- `internal/websterengine/strand_test.go`

**Intent:**
Delete the removers cards 8 and 9 left without callers, so burler and webster each have no stop path but shuttle's (D7).

`remover.go` holds `burlerengine.StrandRemover`, `reedStrandRemover` and `NewReedStrandRemover`; it goes with its test file.
`shedfake.StrandRemover` and its method go, and `shedfake.go`'s file doc comment drops it from the list of fakes; `shedfake_test.go` drops its assertion block.
`websterengine`'s `removeStrandIfLive` goes, `strand.go`'s file doc comment drops its mention, and `strand_test.go`'s `TestRemoveStrandIfLive` goes, its behaviour now pinned by shuttle's stop-verb tests and card 9's stopper assertions.

Before deleting, grep the repository for `StrandRemover`, `NewReedStrandRemover`, `RemoveStrandIfLive` and `removeStrandIfLive`; any reference left outside the deleted code (a doc comment, a doc file) is reworded in this card.

**ImpactSummary:** Removes three unused symbols and one file pair; no remaining caller, so the build and every behaviour stay unchanged.

**Commit:** 10: burler, webster: delete the strand removers the stop verb replaced
