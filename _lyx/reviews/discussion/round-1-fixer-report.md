# Round 1 fixer-report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed: D3's opening now gates only the base strip on the board marker.
  Its second bullet drops former values in every pending worktree strip of a worktree without its own `sparse` marker, in the walk and in `Options.Pair`, whatever the board marker says; its bound is the worktree's own marker.
  Added the acceptance criterion on a clone of a full, unmarked repo and on a mid-merge pair stripped by the next walk.
- F2 (MEDIUM), fixed as proposed, first option: D3 gains a bullet that an absent board marker makes a stamp naming the running build stale, as an absent repo default does.
  The Open risks bullet names every refill path of an older binary and the repair, removing the marker by hand with a board commit; an acceptance criterion covers it.
  The support log's rejected alternative on the refill now names the repair.
- F3 (MEDIUM), fixed as proposed: D1 gains a paragraph naming the weakened Webster-Review and Discussion-Review guards and their bound.
- F4 (MEDIUM), fixed differently: added `## PATTERN changes` with `PATTERN-sparse-hub-base` and updated Q9.
  The proposed entry text said only the walk strip drops a unit, which is false: `lenses` drops template-equal entries on every reconcile (D2) and `ReconcileHubWideAt` removes keys the template lacks.
  The entry says the walk strip is the only one that drops a unit the template holds, bar `lenses`' template-equal entries.
- F5 (LOW), fixed differently: the prime seed's former-value drop is gated on the prime worktree's own `sparse` marker, not on the board marker as proposed.
  The proposed gate broke F1's new criterion: `lyx fabric clone` writes the board marker before its walk, and the walk seeds repo defaults (`reconcileHubWide`, `internal/hubreconcile/walk.go`) before stripping the prime, so a full prime's `discussion_lenses: ""` would have been seeded into the repo default (self-check finding).
  A reseed after migration still keeps a deliberate value, which was F5's premise.
- F6 (LOW), fixed as proposed: D4's key-move bullet states the three rules; the plan note now also names `moveFile.write`'s delete.
- F7 (LOW), fixed as proposed: auto-mode assumption added.
- F8 (LOW), fixed differently: Open risks bullet added, but its way forward is the editor, `lyx config batcher --layer default`, not the proposed `--set profiles.cautious.budget=200000`.
  `splitOpenMapKey` (`internal/yamlengine/set.go:210-216`) takes `cautious.budget` as the entry name, so that `--set` appends a bogus string entry and leaves the budget unchanged (self-check finding).
- F9 (NIT), fixed as proposed: the editor scaffolds an absent hub base with D2's header comment and no unit.

## Self-check

- fix-consistency, MEDIUM: D3's opening still said the strip runs only in the walk, against the `Options.Pair` bullet — fixed as proposed: the opening scopes walk-only to the base strip.
- fix-consistency, LOW: Scope's D3 bullet tied the worktree drop to the walk — fixed as proposed.
- fix-consistency, LOW: the `moveKeys` plan note missed the no-delete rule — fixed as proposed.
- fix-claims, MEDIUM, and fix-way-forward, MEDIUM, merged: the batcher `--set` way forward cannot be carried out — fixed with own change: the editor, since the forks' own alternative is the only working route.
- fix-proposal, BLOCKING: F5's board-marker gate breaks the clone path — fixed as proposed by the fork: gate on the prime's own `sparse` marker.
- fix-proposal, LOW: the marker-removal repair is incomplete against a binary older than the one before this change — fixed as proposed: the risk states the limit.

## Disputed

None.

## Deferred

None.
