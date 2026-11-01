# config --set reaches a key inside a shaped open map, such as a tier env

`lym config gate --set tiers.tier0.env.LYM_TEST_TIER=0` used to land the value one level too high, so the gate config no longer loaded ("cannot unmarshal !!str `0` into gateslot.tierFile").
`--set` wrote only a top-level key or one whole entry of an open map, and the editor refuses a non-interactive terminal, so an orch could not change a key inside a gate tier, such as its `env` or `command`, at all.

Now `--set` takes a dotted path to any key the module's decode reaches below a declared open map, nested maps included, and writes one value there.
A path into a field the shape lacks is refused naming the key and the keys the level does hold.

## Design

- `yamlengine.Shape` describes what a module decodes at an open-map path: a mapping of fixed keys (`Fields`), a mapping of user-owned names (`Entries`), or a single value, optionally `Text`.
  `yamlengine.ShapeOf` derives it by reflection from the Go type the module already decodes into, so the shape follows the struct's yaml tags and cannot drift from the loader.
- `configengine.Spec.OpenMapShapes` maps an open-map path to its shape.
  `SetValues` and `SetSparse` take a new `shapes` parameter, and `configengine.Set` passes the spec's shapes on every layer.
- Missing mappings on the way are created.
  A value on the way that is neither a mapping nor null is an error naming its path.
- Value typing: a `Text` field is always written as a string; a value starting with `{` or `[` is written as a flow mapping or list; any other value is a scalar under the tag its own text resolves to, so `0` is a number.
- Shapes are declared for `gate` (`tiers` as a map of `tierFile`, `build` as `buildFile`) and `drift` (`capture` and `compare` as `stepFile`).
  Other modules' open maps are unchanged and keep today's behavior, since they declare no shape.
- Setting a whole entry or a flow list at an open-map key still works as before; the dotted path adds a finer grain.
- The `lym config` help, the gate and drift templates, and the `configcli`, `configengine`, `driftengine`, `gateslot` and `yamlengine` package docs describe the new form.

## To check

- `drift-open-maps-lack-shape`: the `capture` and `compare` open maps in `internal/driftengine/config.go` now declare a shape; confirm no other module's open map holding mapping entries still lacks one.
- `whole-entry-and-flow-list-branches-untested`: confirm `internal/yamlengine/set_test.go` and the `gateslot` and `driftengine` config tests cover a whole-entry value and a flow-list value written through a shaped path, not only a scalar.
- Not run by hand: `lym config gate --set tiers.tier0.env.LYM_TEST_TIER=0` against a real hub gate file, then `lym gate test`, to confirm the file still loads end to end.
