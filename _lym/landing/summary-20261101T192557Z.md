# lym config --set reaches a key inside a nested map, like a gate tier env

`lym config gate --set tiers.tier0.env.LYM_TEST_TIER=0` used to write a key literally named `tier0.env.LYM_TEST_TIER` directly under `tiers`, one level too high.
gate.yaml then stopped loading (`cannot unmarshal !!str 0 into gateslot.tierFile`).
`--set` treated everything after an open map's path as one entry name, and the editor refuses a terminal-less caller.
So an orch had no way to change one key inside a gate tier or the build step.

## What changes

- **`yamlengine.Shape` and `ShapeOf`.**
  A shape describes the value a module decodes at an open-map path: a mapping of fixed keys (a struct's yaml-tagged fields), a mapping of names (a map), a text value (a string) or a single value (anything else, `yaml.Node` included).
  `ShapeOf` derives it from the decode type by reflection, so the schema is the struct lym already decodes and is never kept as a second list.
- **`SetValues` and `SetSparse` take a `shapes` map** after the per-name maps.
  A key under a shaped open map walks the shape segment by segment and sets that one value, creating every missing mapping on the way.
  A text field is written as a string (`LYM_TEST_TIER: "0"`), any other scalar under the tag its text resolves to (`timeout_min: 30`, not `"30"`), and a value starting with `{` or `[` as a flow mapping or list, as an open-map entry already takes it.
  A key the shape does not reach is refused before anything is merged, and the refusal names the key and the keys the mapping does hold: `unknown config key tiers.tier0.envv.X: tiers.tier0 holds no key envv; it holds about, all_targets, …`.
  A key below a single value, an empty segment, or a non-mapping value on the way is refused the same way.
- **`configengine.Spec.OpenMapShapes`** carries the shapes, and `Set` passes them on every layer: the standalone or seed-only base through `SetValues`, and the sparse hub base and the upper layers through `SetSparse`.
- **Gate declares shapes for `tiers` (`map[string]tierFile`) and `build` (`buildFile`).**
  That makes `tiers.tier0.env.X`, `tiers.tier0.timeout_min` and `build.timeout_min` settable one key at a time.
  `build.timeout_min=5` used to be written as a string, which failed to decode into the `*int`.

## Design decisions

- **Opt-in per open map.**
  An open map with no declared shape keeps today's behaviour, where the dotted rest after the path is one entry name.
  Other modules' open maps, such as board labels or darn's `tree_scope`, may hold names with dots, and declaring a shape for them would change what their keys mean.
- **The shape comes from the decode type.**
  gate.yaml's template declares no tiers (`tiers: {}`), so the template cannot say which keys a tier holds.
  Deriving the shape from `tierFile` and `buildFile` keeps it in step with the loader without a second list to maintain.
- **A one-segment key under a shaped open map still writes the whole entry.**
  `tiers.tier0={...}` writes the entry from a flow mapping, as before.

## Docs

The package docs of `yamlengine`, `configengine`, `configcli` and `gateslot`, the `lym config` help text and the comment on `tiers` in gate's template now describe the nested `--set` form.
To keep the help under its size cap, its open-map example is now the gate one; the board labels example remains in `configcli`'s package doc.

## Tests

- `yamlengine`: `TestSetValues`, `TestSetValues_Refusals` and `TestSetSparse` gain rows for:
  - a key inside an entry, a flow entry included;
  - a fixed-key map writing a number and a text value;
  - an unshaped open map keeping a dotted name;
  - each refusal (missing key, a `-`-tagged field, a key below a single value, an empty segment, an entry holding a scalar).
- `gateslot`: `TestConfigSpec_SetNestedKeys` sets a tier's env and timeout and the build step's command, timeout and env on a hub base through `configengine.Set`, then loads the file with `LoadConfig`.
  It also checks that a misspelled key is refused naming it and leaves the file byte for byte.
