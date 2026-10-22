# Card 1 — configengine-overlay-load

**Create:**
- `plan:internal/configengine#Overlay` -> `type Overlay struct`
- `plan:internal/configengine#EffectiveTemplate` -> `func EffectiveTemplate(spec Spec, base []byte, hub bool) ([]byte, error)`
- `plan:internal/configengine#TemplateUnder` -> `func TemplateUnder(dirs Dirs, spec Spec) ([]byte, error)`
- `plan:internal/configengine#overlayTemplate` -> `func overlayTemplate(spec Spec, value string, hub bool) ([]byte, error)`
- `plan:internal/configengine#selectorValue` -> `func selectorValue(spec Spec, doc []byte) (value string, held bool, err error)`
- `plan:internal/configengine#selectorRefusal` -> `func selectorRefusal(dirs Dirs, spec Spec, path string, heldBefore bool) error`
- `plan:internal/configengine_test#overlaySpec` -> `func overlaySpec() configengine.Spec`

**Edit:**
- `internal/configengine#Spec`
- `internal/configengine#load`
- `internal/configengine#readLayers`
- `internal/configengine#BelowLayer`
- `internal/configengine#Provenance`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine_test#TestLayers`
- `internal/configengine_test#TestProvenance`
- `internal/configengine/doc.go`
- `internal/configcli/doc.go`
- `PATTERN.md`

**Uses:**
- `internal/configengine_test#layerSpec`
- `internal/configengine_test#newLayerFixture`
- `internal/configengine_test#decodeTopLevel`
- `internal/configengine_test#annotationsOf`
- `internal/configengine_test#perNameTail`

**Intent:**
Give `configengine` a template overlay (D4) and make every load-side judging path use the effective template.

`Overlay` is a new struct with two fields: `Selector string`, the name of the top-level template key that picks the overlay, and `Documents map[string][]byte`, each selector value mapped to its overlay document.
`Spec` gains a field `Overlay Overlay`; its zero value means no overlay, and a spec without one loads, fills, warns and annotates byte for byte as today.

`selectorValue` reads the selector's top-level scalar from a document: `held` reports whether the key is present, an absent key or `""` yields `""`, and a non-scalar value is an `ErrInvalid` error naming the key.
`overlayTemplate` is the one composer: with value `""` or no overlay it returns `spec.Template` unchanged, and otherwise it lays `Documents[value]` over `spec.Template` with `yamlengine.Merge` under the spec's open maps, so each overlay name under a per-name map replaces or adds one entry whole.
A value no document names is an `ErrInvalid` error: `<module> config: <selector> "<value>" names no shipped layer (shipped: <sorted names>)`, then the way forward, `run "lyx config <module> --layer default --set <selector>=<name>" from the hub's prime worktree` followed by `boardDirWayForwardClause` when `hub` is true, and `run "lyx config <module> --set <selector>=<name>"` otherwise.
`EffectiveTemplate` is the exported entry point: it reads the selector from `base`, the base layer's document (nil when absent), and calls `overlayTemplate`.
`TemplateUnder` reads the base layer's file of `dirs` (`LayerFile` at `BaseLayer`), treats an absent file as nil, and returns `EffectiveTemplate(spec, data, dirs.Board != "")`; it is how `configsync` judges a non-base file without reading a layer file itself.
No package outside `configengine` composes a template.

`load` takes the effective template from the base layer document in its stack (the base template when the base is absent and the load degrades), and shadows `spec.Template` with it before `warnUnknownKeys`, `FillMissing` and `MissingKeys`, so a name the overlay supplies fills in memory like a template name.
`fallbackToTemplate` is unchanged: with no layer present there is no selector, so the base template is the effective one.
`BelowLayer` starts its merge from the effective template picked by the base layer among the layers it read, for every `layer` argument.

`readLayers` refuses the selector in the worktree file of a hub, where `BaseLayer` is the default layer: after the existing per-layer checks, a worktree doc whose `selectorValue` reports `held` with a value other than the default layer doc's `selectorValue` (`""` when the default file is absent or lacks the key) returns `selectorRefusal(dirs, spec, path, true)`.
A worktree value equal to the base's changes no selection, so it loads, and the sparse migration strips it as equal to the layers below; this is the D3 refinement the decision record carries as an `Added after Discussion` entry, and it keeps a full worktree file written from the template (`lens_set: ""`), such as `hubreconcile`'s pre-migration fixture, loadable and strippable.
`selectorRefusal` returns an `ErrInvalid` error: `config file <path> holds <selector>, which only the base file <base path> may hold; way forward: ` then, when `heldBefore`, `delete the <selector> line from <path> in an editor, then ` and always `set it in the base with "lyx config <module> --layer default --set <selector>=<name>" from the hub's prime worktree`, followed by `boardDirWayForwardClause`.
A local file holding a Guard-classified selector is refused by `ValidateLocal`, a check of the local layer independent of the worktree check; no machine layer stacks for a per-worktree module.
The refusal's bound: a hub worktree file can no longer pick a layer other than the base's, and still overrides any per-name entry.

`Provenance` labels a unit `template:<value>` instead of `template` when no file layer holds it and the picked overlay document holds it, its overlay units taken with `yamlengine.LeafPaths`.

Tests, all through `overlaySpec`: `layerSpec` with `pick: ""` and `perNameTail` appended to its template, `PerNameMaps` `[lenses]`, `pick` and `lenses` classified `CategoryGuard`, and `Overlay{Selector: "pick", Documents: {"x": "lenses:\n  a: X\n  b: B\n"}}`.
`TestLoad_LayerStack` gains an `overlay` row field that swaps in `overlaySpec()` after the loop's spec assignments, so an overlay row keeps `overlaySpec`'s template and `PerNameMaps` whatever its `perNameMaps` field holds (the loop keeps applying `hubWide`, `seedOnly` and `retired`), and a `refusal` row field: the substrings both `Load` and `LoadOrTemplate` must refuse with, marked `ErrInvalid`, the row then asserting nothing else, and a `refusalLacks` field of substrings the refusal must not hold.
Rows: a hub default `pick: x` resolves `lenses` to `map[a:X b:B]`;
a standalone worktree file `fullDefault` plus `pick: x` resolves the same, with `wantFill` `lenses`, the full base's one fill line;
a hub default `pick: x` with `lenses:\n  a: mine` resolves `map[a:mine b:B]`, the file beating the overlay;
a hub default `pick: x` under a worktree file `lenses:\n  b: wt` resolves `map[a:X b:wt]`;
a default without `pick` resolves `map[a:A]`;
a hub default without `pick` under a worktree file `pick: ""` resolves `map[a:A]`, an equal value loading;
a hub worktree file `pick: x` over a default without `pick` is refused with the worktree file's path, `delete the pick line from`, `--layer default` and `the orch runs it`;
a hub default `pick: nope` is refused with `names no shipped layer (shipped: x)`, `--layer default --set pick=<name>` and `the orch runs it`;
a standalone worktree file `pick: nope` is refused with `"lyx config mod --set pick=<name>"`, lacking `the orch runs it`;
a hub default `pick: [x]` is refused with `pick`;
a local file `pick: x` over a hub default is refused with `is guard and cannot be set in a local override`, lacking `only the base file`.
`TestLayers`'s BelowLayer subtest adds an `overlaySpec()` fixture whose hub default holds `pick: x`, and asserts `BelowLayer` at the worktree layer decodes `lenses` as `map[a:X b:B]`, `TemplateUnder` over it decodes `lenses` the same, and `TemplateUnder` over a fixture with no default file returns `overlaySpec().Template` byte for byte.
`TestProvenance` gains an `overlay` row field swapping in `overlaySpec()`, with a row whose default holds `pick: x\n` and wants `pick: default`, `a` and `b` `template:x`, and every other unit `template`.

Docs: `internal/configengine/doc.go` gains a `# Template overlay` section stating the selector, its base-layer source, the composition, the refusals and that every judging path uses the effective template; its Provenance paragraph names the `template:<value>` label.
`internal/configcli/doc.go`'s `--resolved` sentence adds `template:<value>` to the labels it lists.

`PATTERN.md` gains, after the `PATTERN-config-layers` line, the declared entry verbatim:

```
- `PATTERN-template-overlay` — Judging a config document against its template: only against `configengine`'s effective template, the overlay picked from the base layer.
```

The entry adds 171 bytes to an overview already past the headroom line `patternlint` enforces, so this card cuts at least as many, moving detail into background files that already hold it, per `## Shared Decisions` and the record's declared `PATTERN changes`:
the line `Structural invariants, one line per entry: when it applies and what it requires.` and the blank line after it are removed;
`PATTERN-sparse-hub-base`'s text becomes `Writing a hub's base file: only the units the hub tuned, never a template fill.`, its background file stating the walk strip;
and `PATTERN-config-layers`' text becomes ``Resolving a module's config: only `configengine` reads, stacks and locates layer files.``, keeping the backticks the line has today, its background file having a `## Machine layer` section.

**ImpactSummary:** `Spec` gains an optional field every existing spec leaves zero, so every other module's load, provenance and below-layer merge is unchanged; the only new refusals fire for a spec with an overlay, which lenses becomes in card 3.
