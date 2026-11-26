# Card gate runs only the card's listed packages, so a wider intent breaks tests unseen

`card-gate-scope-gaps` · Task · bug, webster, gate, loom

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


## From issue #262

Imported from [issue #262](https://github.com/Knatte18/loomyard/issues/262).

## What happened

A plan had two cards that each add an exported function whose first caller is a later card.
The tier1 guard `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` rejected each at its card gate.
The implementer allowlisted each function in its own commit with a `Loosened-Check:` trailer.
One allowlist entry was still present at the end of the batch, and the card that adds its caller has to remember to delete it, since a stale entry fails the guard.

## Why it is abnormal

An ordinary helper-then-caller card split cannot pass the card gate without loosening a check.
The loosening and its later removal are hand work that no step of the plan states.

## What to do

Either have plan writing and plan review keep a new function and its first caller in one card,
or let the card gate tolerate a function whose caller a later card of the same plan declares.

Provenance: task `config-keys-nested`, note `01-config-key-name.md`.
