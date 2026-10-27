---
round: 2
focus:
  - "Card 8 (_lym/plan/08-dobby-worktree-gone.md), TestRun_WorktreeGoneExits: the loop gains a worktree-gone exit at the top of each iteration and another after each tick before CheckBinary. Settle whether some test row removes the worktree outside a tick, then wakes the loop by one watch event with no probe due, so that deleting the top-of-iteration check would fail a test."
  - "Card 11 (_lym/plan/11-gate-adopt-key.md), seedAdoptGate inside gateslot.MigrateConfig: MigrateConfig also runs for the local-layer file, sparseSeed and configreg.PrepareRunSettings, not only the hub base gate.yaml. Settle whether the card states which of these callers gain the adopt key and whether a test pins that choice."
  - "Cards 8 and 13 (_lym/plan/08-dobby-worktree-gone.md, _lym/plan/13-adopt-check-verb.md): any read the cards add through state.ReadJSON, which runs MkdirAll. Settle whether each such read can recreate a removed directory or write under a path the verb's help text says it leaves untouched."
---

Focus is limited to the gating design finding of round 1, its decision-class neighbour, and the shared read-creates-directory hazard.
