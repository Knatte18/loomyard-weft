# Card 6 — lenses-remove

**Create:**
- `plan:internal/configengine#WarnSkippedEntries` -> `func WarnSkippedEntries(dirs Dirs, spec Spec, key string, entries []string) error`
- `plan:internal/configengine#warnedSkippedEntries` -> `var warnedSkippedEntries sync.Map`
- `plan:internal/lenses#applyRemove` -> `func applyRemove(cfg *Config) []string`

**Edit:**
- `internal/lenses#Config`
- `internal/lenses#ConfigSpec`
- `internal/lenses#Load`
- `internal/lenses#ResolveFan`
- `internal/lenses#TestLoad`
- `internal/lenses#TestCheckLoad`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/lenses/template.yaml`
- `internal/lenses/doc.go`
- `internal/configengine/doc.go`

**Uses:**
- `internal/configengine#StackLayers`
- `internal/lenses#writeLensesYAML`

**Intent:**
Let a repository remove lenses it does not want (D5).

`template.yaml` gains the top-level list key `remove: []`, and its header comment's paragraph saying a template lens or fan cannot be removed is replaced: `remove` lists lens names dropped after resolution from the lenses and from every fan, a fan left empty is dropped, and a name matching no lens is skipped with a warning.
`Config` gains `Remove []string` with tag `yaml:"remove"`, and `EmptiedFans map[string][]string` with tag `yaml:"-"`, mapping each fan the removal emptied to the removed lens names it held, in its order.
`ConfigSpec` classifies `remove` `CategoryGuard`.
The guard's bound: `remove` lets a hub take a review lens out of every fan; as a Guard key no local override or run setting holds it, only a tracked base or worktree file can, a worktree file's list replacing the hub's whole as any list stacks, and a shortened fan still runs its remaining lenses.

`applyRemove` deletes each `cfg.Remove` name present in `cfg.Lenses`, removes every occurrence of those names from every fan, the hub's own fans included, moves a fan that had entries and is left empty from `Fans` into `EmptiedFans`, allocating `EmptiedFans` only when a fan is emptied so a load that empties none leaves it nil, as a decoded fixture compared with `reflect.DeepEqual` holds it, and returns the `Remove` entries that named no lens, in list order.
`Load` calls it after the strict decode, the only place removal applies, and passes a non-empty skipped list to `configengine.WarnSkippedEntries(dirs, ConfigSpec(), "remove", skipped)`, prefixing a returned error `lenses: `.
`WarnSkippedEntries` finds, through `StackLayers`, the highest layer file holding the top-level `key`, and logs one Warn naming the module, the layer, the file, the key and the entries, at most once per process per file, key and entry list, deduplicated in `warnedSkippedEntries` as `warnUnknownKeys` dedups; it returns `StackLayers`' error.
A skipped entry never fails a load, so a lens a later template edit or a layer switch takes away is harmless.

`ResolveFan` checks `EmptiedFans` before its unknown-fan check: a fan held there is refused with `lenses: fan "<fan>" is empty after remove dropped <names>; way forward: point the fan key at another fan or "", or drop a name from remove`.
`CheckLoad` is unchanged: it resolves the fans `Fans` still holds, so a `remove` that empties a fan passes it.

Tests.
`TestLoad` gains rows:
a board `lenses.yaml` with `lens_set: csharp`, `remove: [numerics]` and a hub fan `mine: [numerics, generic]` loads with `numerics` undefined, `review-code` and `full` holding no `numerics`, and `mine` equal to `[generic]`;
a standalone `remove: [no-such-lens]` loads, and a second `Load` of the same anchor leaves exactly one Warn line holding `no-such-lens` and the file's path in the captured log;
a board `remove: [other-missing]` under a worktree file holding only `lenses:\n  mine: prose\n` loads, and its one Warn line names the board file's path and `layer=default`, never the worktree file's.
`TestCheckLoad` gains a row: a standalone file whose `remove` lists the four lenses of `review-code` passes `CheckLoad`, its `Load` lacks `review-code`, and `ResolveFan(cfg, "review-code")` fails holding `remove`, `code-correctness` and `way forward`.
`TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` for `lenses` becomes `{"fans", "lens_set", "lenses", "remove"}`.

Docs: `internal/lenses/doc.go` replaces its "A template lens or fan cannot be removed" sentence with a `# Remove` section stating the above, its `# Guard keys` section names `remove`, and its `# Fan resolution` paragraph names the emptied-fan refusal.
`internal/configengine/doc.go` names `WarnSkippedEntries` beside the unknown-keys warning.

**ImpactSummary:** removal lives in `lenses.Load`, which every lens consumer (burler wiring, loom's twill and fan keys, darn's fan keys) already reads through, so none of them changes; `ResolveFan` gains one refusal reached only through a `Config` that `Load` emptied.
