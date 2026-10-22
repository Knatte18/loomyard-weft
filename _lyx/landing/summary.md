# Batcher profiles merge by name across layers; --set sets one profile key

`batcher.yaml`'s `profiles:` now behaves as a map of names across the layer stack.
A layer's profiles add to or override the template's `identity` and `cautious` by name, and each profile merges key by key.
`lyx config batcher --set profiles.<name>.<key>=<value>` writes that one key.
This fixes issues #833 and #834.

## The faults

`profiles` was declared an open map.
An open map is one unit: the layer merge and the template fill took it whole from the highest layer holding it.
A hub file that added `wide` therefore hid `cautious` and `identity` (#834).
`--set profiles.cautious.budget=300000` split the key at the open map and wrote an entry named `cautious.budget` holding the string `"300000"`.
The load then failed to decode it into a profile (#833).

## The change

- **Batcher** declares `profiles` a per-name map (`ConfigPerNameMaps`), in its spec and in its `configreg` row.
  The layer merge and the template fill already step into a per-name map name by name and into each mapping entry key by key.
  Prune and the unknown-key warning keep a name the template lacks, and reconcile writes only the names a file changed.
- **yamlengine** handles a per-name map whose template entries are mappings.
  `SetValues` and `SetSparse` set `P.<name>` from a YAML flow mapping.
  They know `P.<name>.<rest>` for any name, where `rest` is a scalar or list path some template entry holds, and write that one key under that entry's tag.
  The known-key list names each such path as `P.<name>.<rest>`.
- **configengine** `ValidateLocal` and `ValidateMachine` judge a path under a per-name map that the template lacks against the template entry holding the rest of the path, whatever the name.
  A local or run-settings document can still add a profile, as it could under the open map.
- **Batcher's retired-key refusal** names the highest layer file holding the profile, no longer the layer holding the top-level `profiles` key.
- **configsync**: the former batcher template value narrows to the one key that was tuned, `profiles.cautious.budget: 450000`, since the unit is now a leaf.
  The former-values test checks every unit of the former document.

Two pre-existing YAML emission defects surfaced once `profiles` could be emptied, and both are fixed at their source:

- `FillMissing` appended commented template entries into a flow-style `{}` mapping and emitted invalid YAML.
  A mapping it appends to now takes block style.
- `Reconcile`, `SetValues` and `DropTemplateEqualNames` emptied a per-name map under a key carrying a line comment, and wrote `key: # comment` over a bare `{}` on the next line.
  The comment now moves onto the emptied value, giving `key: {} # comment`.

## Design decisions

- A per-name map instead of a new map kind or a change to open-map merging.
  Changing open maps would alter every module's open maps, including loom's scalar-or-list model keys.
- The first template entry holding a key decides its tag and shape, as the first entry already decided list or scalar entries.
- A layer cannot hide a template profile, since no layer deletes a key from the one below.
  `active:` remains the only selection.

## Effect on existing files

The registry fingerprint covers per-name maps, so hubs reconcile after the binary changes.
Reconcile then drops a profile equal to the template's from a full base, as it does for lenses.
A standalone `batcher.yaml` seeded from the template now reads `profiles: {}`, and the load resolves both template profiles.
The hub's copies of `cautious` and `identity` can be removed by hand, or are dropped by reconcile once they equal the template.

## Tests

- yamlengine: rows for per-name mapping entries in `TestSetValues`, `TestSetValues_UnknownKeys`, `TestSetValues_Refusals` and `TestSetSparse`.
  Regression rows for the commented empty mapping in `TestReconcile`, `TestSetValues`, `TestDropTemplateEqualNames` and `TestFillMissing`.
- configengine: `TestValidateLocal` rows for a profile the template lacks, an unknown key in it and a shape mismatch.
- batcher: `TestActive` rows for an operator profile added beside the template's, one tuned key keeping the template's others, and a tuned key reaching the profile.
  The retired-key row now adds a worktree profile, and still names the repo default.
  Error rows now tune a profile the template lacks.
- configsync: the migration row for batcher now strips every key equal to the layers below and keeps the differing one.
  The open-map-kept-whole case moves to claude's `prompt_cache_ttl_roles`.
