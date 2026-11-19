---
round: 2
focus:
  - "Where `internal/hardenshed` joins `_lym/harden/...` for the round archive and the final ledger, and which root it uses: settle whether the files land under the anchor's `_lym` when the anchor is not `.`, since `fabricengine.CommitAnchoredPaths` takes anchor-relative paths."
  - "How `internal/loomcli/hardenwiring.go` sets the campaign's roots and how the archive deps receive them: settle that the anchor path and the worktree root are not conflated, and that the archive tests use roots that differ."
  - "The Bouncer row built with `harden_round` in `internal/shedrecipe/entries_bouncer.go`: settle that `LiveRed` and `OnHalt` are filled from the campaign and that a test fails if either assignment is deleted, and that the recipe's auto-accept and reviewer-fan setting keys resolve to `hardener.yaml`."
  - "The fix-row wrapper in `internal/hardenshed/round.go` and its test `TestWrapRow`: settle that the archived stray report holds the end-of-row reap's entry with its row and reap point, that the archive runs after that reap, and that a resumed fix row keeps its original starting head."
  - "Any new code from this round's fixes in `internal/hardenshed` and `internal/gatecli`: settle that each fix keeps D7's order of gates, end-of-row reap and archive, and that `--copies` leaves vet-step and guard-step runs single."
---

The focus names the three areas whose fixes carry new design: the archive root, the Bouncer's campaign-filled fields, and the fix row's head and archive order.
