---
round: 2
focus:
  - "Card 2's `_lyx/` probe and `LoadOrTemplate` degrade: with `Dirs{Anchor, Board}` both set and the reading worktree's `_lyx/` absent, does a strict hub-wide `Load` still read the hub file, and does a per-worktree `LoadOrTemplate` still see the repo default? Check cards 10-12 and `lyx fabric reconcile` repair against the same probe rule."
  - "Card 16's migration strip: does any package other than `internal/configengine` read or stack layer files (template, repo default, merge), given the constraint that only configengine does so?"
  - "Card 16's hubreconcile tests: does any new top-level test re-drive a scenario that `internal/hubreconcile/ensure_integration_test.go` already covers, instead of extending the existing test?"
  - "Card 7's retired-key refusal: does the way forward it names actually clear a retired key sitting in the repo default layer, given what `lyx config reconcile --apply` and the hub walk reconcile?"
  - "Cards 4-12 loader tests versus card 13's `TestModules_LocalLayerRefusals`: is any behaviour or per-key refusal pinned by two tests, or a new top-level test created where an existing load test could take the row?"
  - "Card 15's seeding of absent per-worktree repo defaults: is it reachable only from `hubreconcile.Ensure`, or also from `lyx fabric reconcile` and `CloneAndWire`, and does the card state the bound?"
---

Entries name where to look; none caps severity or pre-states a verdict.
