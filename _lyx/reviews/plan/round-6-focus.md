---
round: 6
focus:
  - "Card 15 changes a clone's prime so it carries only the per-worktree config files the records repository already held, and the hubforge weft bare and the clone-test fixtures start empty: sweep every test under any build tag, including integration, that reads `configengine.ConfigFile` or `ConfigFileRel` of a per-worktree module from a hubforge hub or a `CloneAndWire` result without seeding it first, and settle whether each such test is listed under card 15's **Edit:** with its new expectation stated."
  - "Card 15 adds a reseed case to `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` and card 16's `newStaleHub` gives the prime and pair a full per-worktree file set: settle whether the plan names the reseeded module and the case's expectation after the sparse migration, so that `retiredKeyPresent` still reads a non-empty `batcher.yaml`."
  - "Cards 15 and 16 edit the hubreconcile integration tests and the helper `runStaleBuildReconcile`: settle whether each changed test, fake or helper is listed as a member glyph rather than a bare file path."
---

Each entry names where to look and the question to settle.
