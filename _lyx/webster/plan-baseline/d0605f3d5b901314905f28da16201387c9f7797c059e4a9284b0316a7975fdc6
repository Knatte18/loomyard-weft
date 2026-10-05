# Card 5 — lyx config --set reaches open maps

**Edit:**
- `internal/configcli#setModule`
- `internal/configcli#buildConfigLong`
- `internal/configcli/configcli_test.go`
- `docs/overview.md`

**Create:**
- `internal/configcli/doc.go`

**Uses:**
- `internal/configreg#Lookup`
- `internal/configengine#Set`

**Intent:**
`setModule` looks the module up with `configreg.Lookup` and passes its `OpenMaps` to `configengine.Set`, so `lyx config board --set labels.quarry="glyphs and the quarry index"` adds or rewrites one label (D3).
`buildConfigLong` documents setting one key of an open map with that example, that removing a key stays an editor edit, and that a list-shaped value must be rewritten as a map first.

Bound: only paths the module declares open reach the new behaviour (card 1's rule), `--set` under one writes scalar values only, and an undeclared nonexistent key still refuses.
What it lets through: a mistyped `<name>` under `labels` creates a new label instead of refusing.

Tests, untagged through `dispatch`: `--set labels.x=desc` on a map-shaped `board.yaml` writes `x: desc` under `labels`; on a list-shaped one it refuses and writes nothing; `--set` on an undeclared nonexistent key still refuses and never syncs.

`doc.go` gains the package doc for `configcli`: the edit, `--print`, `--set` and `reconcile` paths.
`docs/overview.md`'s `lyx config <module> --set` line names open maps and the label example.

**ImpactSummary:** the `--set` path of `lyx config`; no other config verb changes.

**Commit:** 5: lyx config --set reaches open maps
