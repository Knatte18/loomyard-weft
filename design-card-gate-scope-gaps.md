# Card gate runs only the card's listed packages, so a wider intent breaks tests unseen

`card-gate-scope-gaps` · Note · bug, webster, gate, loom

Imported from [issue #263](https://github.com/Knatte18/loomyard/issues/263).

## What happened

A card's stated intent covered every remaining message of one kind across the repository, but its edit list named a fixed set of packages and its card gate ran only those.
Tests outside the list that assert on the changed messages failed unseen by the gate (in `burlercli`, `shuttleengines`, `configcli` and `loomengine`).
The implementer found them only by testing extra packages by hand.

A related miss in the preceding card: a key the card attributed to `loomengine` and `loomcli` is set in `internal/shedrecipe`, which was not among the card's targets.

## Why it is abnormal

The card gate passes while the change breaks tests, so the failure surfaces later or not at all within the card.
Plan writing and plan review accepted an intent broader than the card's targets.

## What to do

- Have the card gate also run the packages whose tests depend on the changed packages, or on the changed strings.
- Have plan review reject a repository-wide intent whose edit list is a fixed set of packages with no step that proves the set complete.

Provenance: task `config-keys-nested`, notes `11-module-key-messages.md`, `10-loom-key-messages.md`.
