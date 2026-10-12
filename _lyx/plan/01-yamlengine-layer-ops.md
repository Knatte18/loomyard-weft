# Card 1 — yamlengine layer operations

**Create:**
- `internal/yamlengine#Merge`
- `internal/yamlengine#Strip`
- `internal/yamlengine#SetSparse`
- `internal/yamlengine#Prune`
- `internal/yamlengine#TestPrune`
- `internal/yamlengine#LeafPaths`
- `internal/yamlengine#AnnotateLeaves`
- `internal/yamlengine#TestMerge`
- `internal/yamlengine#TestStrip`
- `internal/yamlengine#TestSetSparse`
- `internal/yamlengine#TestLeafPaths`
- `internal/yamlengine#TestAnnotateLeaves`

**Edit:**
- `internal/yamlengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the six node-level operations the layer stack, the sparse reconcile, the sparse writes, the migration strip and the provenance print need, all over raw unexpanded YAML bytes, in a new file beside `fill.go` (the implementer names it).
They share one notion of a *unit*: a mapping key is descended into, except a key at a declared open-map path and, when `wholeEntries` is true, every top-level key, each of which is one unit carried or compared whole; a scalar or a sequence is always one unit.

- `Merge` returns `upper` laid over `lower` key by key at every mapping depth: a unit `upper` holds replaces `lower`'s, a unit only `lower` holds stays, and a mapping both hold merges recursively.
  No layer can delete a key from the one below.
  An empty or comment-only side yields the other side's bytes; both empty yields empty bytes.
  Comments and key order come from `lower`, with `upper`'s new keys appended in `upper`'s order.
- `Strip` removes from `upper` every unit whose node equals `lower`'s node at the same path, and returns the stripped bytes and the sorted dotted paths it removed.
  Equality is over the unexpanded nodes: same kind, same scalar tag and value, same children, comments ignored; so a `${env:...}` marker equals only the same marker text, in either direction.
  A mapping left empty by the strip is removed with its key; an `upper` left with nothing returns empty bytes.
- `SetSparse` applies `pairs` to `existing` (an empty mapping when `existing` is empty or comment-only) and writes only the set keys, creating intermediate mappings; it never scaffolds from the template.
  Key knowledge, open-map entries and flow-list values follow `SetValues` exactly, and `SetResult.Unknown`/`Known` are filled the same way, with `Merged` nil when `Unknown` is non-empty; `Preserved` stays empty.
- `Prune` removes from a sparse `existing` every unit the template does not know, a key under a declared open map always known, and returns the pruned bytes and the sorted removed paths; it never adds a key, so it is a sparse layer's counterpart of `Reconcile`.
  A mapping left empty is removed with its key, and an `existing` left with nothing returns empty bytes.
- `LeafPaths` returns the sorted dotted paths of every unit in `doc`.
- `AnnotateLeaves` writes each `notes[path]` as the line comment of the unit at `path` and returns the re-marshalled document; a path `doc` lacks is ignored.

The tests are table-driven at the package surface, one per operation: `TestMerge` pins depth merge, scalar and list whole replacement, open-map and whole-entry carry, and the empty-side rows; `TestStrip` pins both marker directions, an open map equal except one entry kept whole, an emptied nested mapping removed and the all-stripped empty result; `TestSetSparse` pins a set into empty bytes writing only that key, a set beside an existing key, an open-map entry, a list value and an unknown key; `TestPrune` pins a retired key removed, an open-map entry kept, no template key added and the emptied result; `TestLeafPaths` and `TestAnnotateLeaves` pin units under both flags.
`doc.go` gains one line per operation and the unit rule.
The budget row for `internal/yamlengine` rises by the six new tests.

**ImpactSummary:** additive: six new exported functions in yamlengine and its package doc; no existing caller changes.
