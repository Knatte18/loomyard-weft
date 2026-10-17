# shedrecipe: per-segment settings and one told setting lookup instead of per-value Env fields

`shedrecipe-capabilities` · Task · enhancement, shed, loom, config · running · Merge-In

From `manifest/roadmap.md` Someday.
Giving a producer a new capability means hand-threading a passthrough `Env` field through three layers, because the Shed Recipe Registry entry (`PATTERN-shed-recipe-registry`) bars `shedrecipe` from importing the capability's owning package.
The idea, not yet designed: let a producer declare what it needs and have the registry wire it.
Deep, likely touching the invariant itself and every registry entry.

## Seen (2026-10-17)

`shedrecipe.Env` (`internal/shedrecipe/recipe.go`) has 80 fields.
Since 2026-10-01, 38 of 289 commits changed `recipe.go`, adding about 64 `Env` fields.
Each new config value touches four places: the wiring in `internal/loomcli/wiring.go`, the `Env` field, the row that reads it (for example `internal/shedrecipe/entries_burler.go`), and the field list in `internal/testkit/envkit/envkit.go`.
Example: the fix-fan list goes `loom.yaml` -> `wiring.go` `RowFixFans` -> `Env.RowFixFans` -> `entries_burler.go`.
`PATTERN-told-geometry` keeps engines told rather than deriving, so they run alike in a hub, in a standalone CLI (`standalonegeom`) and in a bare test directory; that bound is about paths, while the per-value threading comes from `shedrecipe` neither reading config files nor importing `loomcli`.

## Operator decision 2026-10-17

Build the first two steps as one loom task, not the full capability declaration:

1. Per-segment settings instead of per-value maps: fold the per-row maps (`RowClusterFans`, `RowFixFans`, the per-row reviewer and fixer model lists and their kin) into one `Env.Segments map[string]SegmentSettings`, so a new value is one struct field and a new review segment is data, not new `Env` fields.
2. One told lookup instead of many fields: `Env` gains one caller-filled closure, such as `Setting(module, key)`, and a recipe row names the key it reads, as `FixFanKey: "fix-fan"` already does, so a new config value needs a recipe key and a template key and no Go wiring change; `shedrecipe` still never learns the file format.
   A recipe that names an unknown key is refused when it loads, since a string lookup loses the compile-time check.

The standalone and test bound stays: the hub fills the lookup from its module config, and a standalone CLI or a test fills it with its own values.
It runs after the code-review fan keys and the loom model key renaming have landed, since both change the same wiring and key names.
