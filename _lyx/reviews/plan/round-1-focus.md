---
round: 1
focus:
  - "Fidelity, D1: the decision names every weft `git worktree add` path, including `add`'s new origin-adopt path from D7. Settle whether card 1's `ensureWeftLockDirAt` call in `Topology.Add` and card 4's resolver-created-branch adoption reach the same `containedWorktreeAdd` call, so no adopt path in `Add` is left without `.weft/`."
  - "Fidelity, D6 and D7 against cards 3 and 4: settle whether every bullet of D6's live-pair rule (archive tags consulted only without a local weft, SkipGit/SkipPush, fast-forward, record preservation, rollback scope) and D7's reconcile and checkout bounds is carried by a card, and whether any card step goes beyond what D5 to D7 license."
  - "Granularity: settle whether card 1 (creation at birth, reconcile heal, `ReadOriginFor` error, three test files across two packages) and card 4 (probes, creation order, record step, rollback, five test files) are each one independently reviewable unit, or whether a card bundles parts that should be reviewed separately."
  - "Test economy: for each new test file and row (`weftlockdir_integration_test.go`, the `reedgeom_integration_test.go` rows, `adoptremote_integration_test.go` rows, `configchanges_integration_test.go`), settle whether an existing test already covers the behavior or whether two cards plan overlapping coverage of the same path, such as card 1's `lyx fabric add` rows against card 4's add-adopt rows. Name the covering test in any finding."
  - "Attack-surface question: for card 2 (origin consulted before fork), card 3 (archive-guard bypass), card 4 (replaced warp divergence refusal, fast-forward of a local branch), card 1 (reconcile now writes `.weft/`) and card 6 (new `Notify` seam), settle whether each `**Intent:**` states what the change can now skip or let through and what bounds it."
  - "Verify coverage: settle whether `## verify:` runs the tests of every package any card targets, including the untagged, `integration` and `tmux` tiers the cards' new tests sit in."
  - "Design, sequencing: settle whether the card order holds, given that card 3 and card 4 use the resolver card 2 creates, that cards 1 and 4 both edit `Topology.Add`, and that card 6 depends on card 5's `ReadConfigChanges`."
---

Focus entries name where to look; none is a verdict.
No entry re-derives a mechanical check of the plan spec, and none asks for derived `DependsOn`/`Produces` fields.
