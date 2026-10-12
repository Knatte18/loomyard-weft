---
round: 4
focus:
  - "Card 16's `Ensure` integration tests in `internal/hubreconcile` (stale-build reconcile, mid-merge pair retried, merge-beginning-before-commit restore): the fixture is `newStaleHub`, which is `CopyHub` plus `AddPair` on a hubforge template hub, and hubforge builds hubs through `CloneAndWire`, which card 15 makes end by calling `hubreconcile.Ensure`. Settle whether each worktree the fixture hands the test is unmarked and holds full per-worktree files at the moment the walk under test starts, so each stated assertion (stripped and marked, skipped pair stripped by retry, no marker after restore) can hold."
  - "What a clone produces once cards 15 and 16 have both landed: the `internal/hubforge/doc.go` seeding contract, the mutation record `internal/fabriccli#CloneAndWire` returns, and the `CloneConfigCommit_MutationRecordShape` step of `TestRunCLI_CleanHubScenario`. Settle whether each plan card that touches them describes the same clone outcome, including any committed sparse marker and the seeded defaults."
  - "Card order and fixture reuse across cards 15 and 16: settle whether any card relies on a hub state (marker present or absent, per-worktree files present or absent) that an earlier card's `CloneAndWire` change has already altered."
---

The round-3 review's findings concern the interaction of cards 15 and 16 around the clone's `Ensure` walk; round 4 should check that the fixes there are coherent and that nothing else in the plan assumes the old clone outcome.
