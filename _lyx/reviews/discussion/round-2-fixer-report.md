# Round 2 fixer report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed.
  Premise checked: `modelspec.LoadRegistry` replaces each alias whole, with "no field-level merge" (`internal/modelspec/load.go:54`), and `knownParams` admits `version` in `Entry.Defaults` (`internal/modelspec/modelspec.go:121`).
  D1 adds that each top-level `models` alias is carried whole from the highest layer holding it and inherits none of the lower entry's `defaults`.
  D2's merge sentence now says the same instead of key by key.
  D6 states a `models` alias is compared and stripped as one unit.
  A new acceptance row pins that a local alias without a `defaults` block resolves with no defaults from the layers below.
- F2 (LOW), fixed as proposed.
  D5 exempts a key path under any top-level alias in the `models` local file, whether or not the template holds that alias, with the models loader validating the block at load.
  D8's validation clause and the `lyx board upsert` acceptance row say a key path under any `models` alias.
  The acceptance row for `models` now also accepts `opus.defaults.version` under an alias the template holds.
- Support log: a round 2 rerun entry under Review rounds, the key-by-key alias merge under Rejected alternatives, and Q9 updated.

## Disputed

None.

## Deferred

None.
