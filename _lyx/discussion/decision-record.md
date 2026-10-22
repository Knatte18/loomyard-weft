# Decision record — config layers set one profile key and merge profile maps by name

## Goal

`batcher.yaml`'s `profiles:` behaves as a map of names across the layer stack:
a layer's profiles add to or override the template's by name, each profile's keys merging leaf by leaf,
and `lyx config batcher --set profiles.<name>.<key>=<value>` sets that one leaf.

## Scope

- Issue #833: `--set profiles.cautious.budget=300000` is refused with an unmarshal error, because `profiles` is declared an open map and the set writes the whole rest of the key, `cautious.budget`, as one entry name.
- Issue #834: a layer holding `profiles:` hides the template's profiles, because an open map is carried whole by the merge and by the template fill.
- In scope: batcher's spec and registry row, yamlengine's per-name set for a map whose entries are mappings, configengine's local and machine checks for a name the template lacks under a per-name map, batcher's per-profile origin for the retired-key refusal, and the docs of the three packages.
- Out of scope: the hub's own `batcher.yaml` copies of `cautious` and `identity`, which live on Bolt and are the operator's to remove; reconcile drops them once they equal the template.

## Decisions

### Batcher declares `profiles` a per-name map, no longer an open map

Decision: `batcher.ConfigSpec` and the `configreg` row carry `profiles` in `PerNameMaps` and no open map; `ConfigOpenMaps` gives way to `ConfigPerNameMaps`.
Rationale: the layer merge and the template fill already step into a per-name map by name and into each mapping entry key by key, prune and the unknown-key warning keep a name the template lacks, and reconcile writes only the names a file changed.
That is the behaviour both issues ask for, so the declaration fixes the load side with no new merge rule.

### A per-name map whose template entries are mappings takes a flow mapping and entry leaves on `--set`

Decision: in `yamlengine.SetValues` and `SetSparse`, a per-name map whose template entries are mappings sets `P.<name>` from a YAML flow mapping, replacing the entry whole, and knows `P.<name>.<rest>` for any name where `<rest>` is a scalar or list path of some template entry at P.
Such a pair writes that one leaf, creating the name's mapping when absent, under the tag of the template entry's leaf, as a template leaf is written.
The known-key list names each such path as `P.<name>.<rest>`.
Bound: only a path some template entry holds is opened; a deeper key no entry holds stays unknown and refused, and per-name maps of scalars or lists keep their dot-free rule.

### The local and machine checks judge a new name under a per-name map against the template's entries

Decision: `ValidateLocal` and `ValidateMachine` judge a unit `P.<name>.<rest>` whose name the template lacks against the first template entry at P holding `<rest>`, for existence and shape; categories decide as before.
Rationale: under the open-map declaration a local or run-settings document could add a profile; without this, the per-name declaration would refuse a profile the template lacks there.
Bound: existence and shape are still checked, and the category of P still decides, so a per-name map of a guard category, such as lenses' and gate's, stays closed to both layers.

### The retired-key refusal names the highest layer holding the profile

Decision: `batcher`'s loader records for each profile the highest layer file holding `profiles.<name>`, in place of the layer holding the top-level `profiles` key.
Rationale: a profile now merges from several layers, so the top-level origin can name a file that holds no part of that profile.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads, stacks and locates layer files; batcher reads origins from `StackLayers`' layers.
- `PATTERN-hub-wide-config` and `PATTERN-sparse-hub-base`: a hub base stays sparse; a set writes only the pairs.
- `PATTERN-binary-change-reconcile`: the registry fingerprint covers per-name maps, so the declaration change reconciles hubs through `hubreconcile.Ensure` with no new code.
- `PATTERN-tolerant-records` does not apply: YAML config, not JSON records.
- `PATTERN-comment-line-breaks` and the docs rule in `CLAUDE.md`: `doc.go` of `configengine`, `yamlengine` and `batcher` change in the same commit.

## Auto-mode assumptions

- A per-name map holds one kind of entry, so its first template entry decides scalar, list or mapping, as for lists today.
- Reconcile dropping a template-equal profile from a full standalone `batcher.yaml` is wanted: the load resolves it from the template, as for lenses.

## Open risks

- A hub `batcher.yaml` that relied on hiding a template profile can no longer do so; no layer deletes a key from the one below, so `active:` choosing the profile it wants is the only selection.
- A retired key in a lower layer's part of a profile split across layers is attributed to the highest layer holding that profile.

## Acceptance criteria

- `--set profiles.cautious.budget=300000` on the default layer of a hub writes only that leaf, and the resolved config keeps every other `cautious` key from the template.
- `--set profiles.wide.budget=250000` is known and writes that one leaf as an integer; `--set profiles.wide={batchifier: identity}` writes the entry whole; `--set profiles.wide.nope=1` is refused as unknown.
- A hub file holding only `profiles.wide` resolves `wide`, `cautious` and `identity`, and `batcher.Active` with `active: cautious` builds the template's cautious batchifier.
- A layer overriding one leaf of `cautious` resolves that leaf from the layer and the rest from the template.
- A local document adding a profile the template lacks passes `ValidateLocal`, and one adding a key no template profile holds is refused.
- The retired-key refusal names the layer file holding the profile.
- Existing per-name map behaviour for lenses and gate is unchanged.

## Notes for the plan writer

- `yamlengine.listPerNameMapPaths` reads the first entry's kind; generalising it to an entry kind covers the mapping case.
