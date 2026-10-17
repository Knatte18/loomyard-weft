# Card 1 — configreg-setting-lookup

**Create:**
- `plan:internal/configreg#SettingLookup` -> `func SettingLookup(dirs configengine.Dirs) func(module, key string) (any, error)`
- `plan:internal/configreg#TestSettingLookup` -> `func TestSettingLookup(t *testing.T)`

**Edit:**
- `cmd/lyx#configStrictnessStrictSet`
- `pattern/PATTERN-config-strictness.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add D4's hub-side setting lookup to `internal/configreg`.
`SettingLookup(dirs)` returns a read-only closure of the plain func type `func(module, key string) (any, error)`, the type `shedrecipe.Env.Setting` will take in card 3, naming no `shedrecipe` type.
On a call it:

1. resolves `module` with `Lookup`; an unregistered module is an error naming the module and pointing at `lyx config` (the registered module names), e.g. `configreg: setting module "lom" is not a registered config module; way forward: name one of <Names()>`;
2. checks `key` against the top-level keys of the module's embedded template (`Module.Template()` decoded with `yaml.v3`), never against the loaded document, because `configengine.Load` fills the document down to the template; a key the template lacks is an error naming the module and key, with the way forward to name a key of `<module>.yaml`'s template;
3. loads the module's stack once with strict `configengine.Load(dirs, m.Spec())`, decodes it into `map[string]any`, and keeps that snapshot in the closure, so every later key of the same module answers from it; a `Load` or decode error is returned wrapped with the module name;
4. returns the snapshot's value for `key` unchanged (`nil` for a YAML null, `string`, `bool`, `int`, `[]any` as `yaml.v3` decodes them); shape checking is the reader's job.

`Load` already applies `PATTERN-config-layers`' local-layer bounds (it refuses a local file holding a guard-category key), so the lookup never answers a guard key from a local or `run_settings` layer.
A hub-wide module such as `darn` resolves from `dirs.Board`, because `Load` reads the spec's `HubWide` flag; the caller passes the same `Dirs` its module loaders use.
A mutex guards the snapshot map so the closure is safe for concurrent reads.
Bound: the closure only reads, answers only template keys of registered modules, and loads each module at most once per closure; a caller wanting to see a changed file makes a new lookup.
The doc comment on `SettingLookup` states this bound.

`configreg` now calls `configengine.Load`, so `configStrictnessStrictSet` in `cmd/lyx/configstrictness_test.go` gains `"internal/configreg": true`, and the strict-caller list in `pattern/PATTERN-config-strictness.md` gains `configreg` with one line saying it loads strictly to answer a recipe row's setting.

`TestSettingLookup` is a table whose every row builds its own temp hub fixture (`t.TempDir()` under the package's existing `TestMain`), so the rows that rewrite or corrupt a file share no state with the others and run in parallel: a board dir and an anchor dir, each with a `_lyx/config/` directory, the loom default file written at `configengine.ConfigFile(<board _lyx dir>, "loom")` from `loomengine`'s template text with `review_max_bounces` changed, and a darn hub file at the board.
Rows, each asserting the exact value or an error containing the module, the key and `way forward`:
- a key no layer sets answers its template default (`loom`, `plan_auto_accept` → `false`);
- a key the default file sets answers the file's value;
- a `CategoryNone` key (`review_max_bounces`) set in the pair's local layer at `configengine.LocalFile(anchor, "loom")` answers the local value;
- a guard-category key (`plan_auto_accept: true`) written into the local layer makes the call fail rather than answer the local value, so a guard key never takes a local value;
- one lookup answers a second key of a module from its snapshot: after the first read the test rewrites the default file's value, and a second read through the same closure still answers the first value, while a fresh `SettingLookup` answers the new one;
- the hub-wide `darn` module answers `darn_fixer_fan` from the board's darn file;
- an unregistered module is refused;
- a key absent from the template is refused, including a key only the on-disk default file holds;
- an unreadable layer propagates as an error: the default file holding invalid YAML makes the call fail naming the module.

`cmd/lyx/testdata/test-budget.yaml`'s `internal/configreg` row goes from 10 to 11 for the new top-level test.

**ImpactSummary:** One new exported configreg function with no caller yet, one pinned strict-set entry in cmd/lyx's scan and a doc line; no existing behavior changes.

**Commit:** 1: configreg: add SettingLookup, a snapshot lookup of a module's resolved config key
