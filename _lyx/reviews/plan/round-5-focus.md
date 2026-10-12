---
round: 5
focus:
  - "Card 15 (`_lyx/plan/15-configsync-layer-reconcile.md`) changes what `CloneAndWire` writes into the prime, so a clone's prime carries no per-worktree config file and no `fabric clone: record module configs` commit when the weft template holds none. Settle whether every existing test that reads a per-worktree config file from a hubforge hub or a clone, across the whole module including tests under the `integration` tag, is either listed under card 15's or a later card's **Edit:** with its new expectation stated in **Intent:**, or seeds its own file first."
  - "Card 16 (`_lyx/plan/16-sparse-migration.md`) builds `newStaleHub` for the `Ensure` integration tests in `internal/hubreconcile`. Settle whether each test starts from an unmarked worktree with full per-worktree files and a retired key still present, and whether the file-set writes and the `seedRetiredKey` call are ordered and listed so `retiredKeyPresent` finds a non-empty `batcher.yaml` after a walk."
  - "Card 16's `Ensure` tests that assert a second or next walk (the restore-after-merge test and the stale-build reconcile helper): settle whether the state each one sets up (merge record cleared, stamp removed) lets that second walk actually run, so its assertion tests something."
---

Focus targets the interaction of cards 15 and 16 with tests outside their own scenarios.
