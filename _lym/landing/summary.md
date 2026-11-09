# Loom config tests: a new key no longer breaks every load case

Adding loom config keys failed dozens of tier 0 tests, nearly all of them cases of `TestLoadConfig_Loads` in `internal/loomengine`.
Each case compared the whole loaded `Config`, so one new key broke every case.
This change makes those cases assert only what they change, and makes the one test that must fail for an unclassified key say how to fix it.
Only test files change.

## What changes

- `TestLoadConfig_Loads` builds each case's expected `Config` from the template's own load (`loadTemplateConfig`), then applies the case's `mutate`.
  A key added to the template needs no edit to the table.
- The `templateConfig()` struct literal is replaced by `templateDefaults`, a map from field name to the value the template's line parses to.
  A `template defaults` subtest checks each entry against the template load, so the parse of each existing key stays pinned.
  A key added to the template needs no entry; an entry naming a field `Config` no longer has fails the subtest.
- `TestModules_EveryTemplateKeyClassified` in `internal/configreg` stays the one test that pins each module's whole key set.
  Its failures now name the key and where it goes:
  - an unclassified template unit names the `Categories` map (by its function) to add it to, and the two categories to choose between;
  - a `Categories` entry with no template path says to drop it or add the template line;
  - a `LocalKeys`, `LocalDenied` or `MachineKeys` mismatch names the keys to add to or drop from the expected list, via `keyListWayForward`.
- `TestKeyListWayForward` pins the wording of that helper for add, drop, add-and-drop and order-only cases.

## Other whole-set tests

The task asked to list the other tests that fail for any added key.
They are `TestLookup` in `internal/configreg`, `TestWire_ReviewKeysReachTheEnv` in `internal/loomcli` and `TestRecipes_KeySet` in `internal/shedcli`.
This change leaves them as they are: each pins a key set on purpose, so a new key failing them is the intended prompt to register it.

## Design decisions

- The expected value of a case is the template's load, not a fixed literal.
  A case that changes nothing from the template therefore compares the template load with itself; the `template defaults` subtest is what guards the template's own values.
- The whole-key-set pin stays in `configreg` rather than being removed, because a key nobody classified must still fail.

## To check by hand

Nothing outstanding; the change is tests only and the tier 0 tests for `internal/configreg` and `internal/loomengine` cover it.
