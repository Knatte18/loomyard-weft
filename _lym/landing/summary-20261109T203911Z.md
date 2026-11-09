# A new loom config key no longer breaks every TestLoadConfig_Loads case

`TestLoadConfig_Loads` in `internal/loomengine` compared each case against a hard-coded whole `Config`.
Adding one key with a non-zero default to `loom.yaml` therefore failed every case, about fifty subtests.
This change makes each case pin only the fields it is about.
It also makes the one test that pins every module's key set say which key goes where.

## What changes

- **`internal/loomengine/config_test.go`**: the hard-coded `templateConfig()` struct is gone.
  The test loads the shipped template once, and each case's `mutate` changes that loaded value.
  A case therefore names only the fields its file changes, and every other field must equal the template's own load.
  The hub-stack subtests use the same base.
  The template's parsed values are still pinned by a `template defaults` subtest.
  That subtest compares each field named in a `templateDefaults` map, keyed by `Config` field name.
  A key added to the template needs no entry, and a renamed field fails, naming the stale entry.
- **`internal/configreg/configreg_test.go`**: `TestModules_EveryTemplateKeyClassified` stays the one test that pins each module's whole key set.
  Its messages now name what to do:
  - An unclassified template unit names the key and the `Categories` map to add it to.
    The map's owner is the module's `Spec` function, found through `runtime.FuncForPC`, so the message cannot go stale.
  - A `Categories` entry that names no template path says to drop it, or to add the template line.
  - A `LocalKeys`, `LocalDenied` or `MachineKeys` mismatch lists the keys to add to and drop from the expected list in the test.

## Design decisions

- The base each case is compared against is the template's own load, not a struct literal.
  A case still asserts that nothing beyond its named fields moved.
  The table no longer restates the default of every key.
- Template values are pinned field by field through a name map, not a struct literal.
  So the pin holds for a zero default such as `DiscussionInteractive: false`, and a new field never touches it.

## Other tests that pin a whole struct or key list

These fail when a key is added. They are listed here and left unchanged, as the board entry asks.

Any key with a non-zero default in that module's template breaks:

- `internal/darnengine/config_test.go` `TestLoadConfig` (`templateConfig()` literal)
- `internal/driftengine/config_test.go` `TestLoadConfig` (`templateConfig()` literal)
- `internal/dobbyengine/config_test.go` `TestLoadConfig` (`defaultsWith()` literal)
- `internal/gateslot/config_test.go` `TestLoadConfig` and `TestTemplateConfigAndLimits` (`templateDefaults` struct, whole `Gates` map included)
- `internal/websterengine/config_test.go` `TestLoadConfig` (`templateDefaults` and `absentKnobs` literals)
- `internal/sheddriver/config_test.go` `TestLoadConfig` (`templateDefaults` and full `Config` literals)
- `internal/samplerengine/config_test.go` `TestLoadConfig`
- `internal/slackengine/config_test.go` `TestLoadConfig`
- `internal/reportconfig/config_test.go` `TestLoadConfig`
- `internal/loomengine/tapestry_test.go` `TestLoadTapestryConfig` (full `TapestryConfig` literal; any `tapestry.yaml` key)

Any key at all, by design:

- `internal/slackengine/config_test.go` `TemplateKeysAreExactlyRemindAfterMin` pins slack's whole key list.
- `internal/configreg/configreg_test.go` `TestModules_EveryTemplateKeyClassified`, the classification pin kept above.

Only a key of a certain kind:

- `internal/configreg/configreg_test.go` `TestLookup`: a new list-valued loom or tapestry key, through the pinned `OpenMaps` list.
- `internal/loomcli/wiring_test.go` `TestWire_ReviewKeysReachTheEnv`: a key that adds a review segment or a `SegmentSettings` field, since each segment's entry is pinned whole.
- `internal/shedcli/table_test.go` `TestRecipes_KeySet`: a new recipe, not a config key.
- `internal/lenses/config_test.go` `TestConfigTemplate_DecodesAndIsSelfConsistent`: a new lens or fan.
- `internal/modelspec/template_test.go` `TestConfigTemplate_LoadsWithLiveEntries`: a new model alias.

The `internal/configengine` and `internal/yamlengine` tests were not surveyed.
