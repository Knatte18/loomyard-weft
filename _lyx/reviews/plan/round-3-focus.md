---
round: 3
focus:
  - "Seeding of per-worktree repo defaults in a fresh hub: does any card before card 9 (where loom loads strictly through `fabricengine.ConfigDirs`) leave a `fabriccli.CloneAndWire` or `hubforge` hub with a repo default under the board dir, and is the seed kept inside `hubreconcile.Ensure` (`PATTERN-binary-change-reconcile`)? Check that `internal/hubforge/doc.go`'s seeding contract is edited by a card if the plan changes what a clone materialises."
  - "Refusal way forward for an absent base layer under the board dir in card 2: for a per-worktree module, does the named step actually create the file, given what card 15's `ReconcileAll` and `lyx fabric reconcile` do with an absent default (`PATTERN-refusal-way-forward`)? Check the `TestLoad_Refusals` row pins the same text."
  - "Layer stacking location: does any card other than `configengine` (cards 7 and 8, `burlerengine.LoadConfig`, `modelspec.LoadRegistry`, batcher's retired-key attribution) merge or order layer files itself, against the constraint that only `configengine` reads or stacks layer files and forms their paths, which card 13 writes into `PATTERN.md`?"
  - "Test economy across loader cards 4, 5, 8, 9, 10 and card 13's `TestModules_LocalLayerRefusals`: is any local-layer key's refused or accepted verdict pinned by both a loader card's wiring row and a row of card 13's table (`PATTERN-test-economy`)? Name the shared key and the two sites."
  - "Card 16's failed-walk behaviour: does the plan make `hubreconcile`'s config snapshot and restore cover the sparse marker, so a worktree failed before commit stays unmarked as the card's own assertion states?"
---

Round 3 focus targets the open round 2 findings; each entry restates its sites so the reviewer needs no earlier round.
