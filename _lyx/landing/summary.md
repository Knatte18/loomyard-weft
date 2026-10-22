# Config layers set one profile key and merge profile maps by name

`batcher.yaml`'s `profiles:` now behaves as a map of names across the layer stack.
A layer's profiles add to or override the template's by name, each profile's keys merging leaf by leaf.
`lyx config batcher --set profiles.<name>.<key>=<value>` sets that one leaf.

Before, `profiles` was declared an open map.
The layer merge and the template fill carried an open map whole, so a hub layer holding `wide` hid the template's `cautious` and `identity`.
`--set profiles.cautious.budget=300000` was refused with an unmarshal error, because the set wrote `cautious.budget` as one entry name.

Closes #833
Closes #834

## What changes

- `batcher.ConfigSpec` and its `configreg` row declare `profiles` in `PerNameMaps` and no open map; `ConfigOpenMaps` gives way to `ConfigPerNameMaps`.
  The merge and fill already step into a per-name map by name and into each mapping entry key by key, so this fixes the load side with no new merge rule.
- `yamlengine.SetValues` and `SetSparse` take a per-name map whose template entries are mappings:
  - `P.<name>` sets from a YAML flow mapping and replaces the entry whole.
  - `P.<name>.<rest>` is known for any name where `<rest>` is a scalar or list path of some template entry at `P`.
    It writes that one leaf under the tag of the template leaf and creates the name's mapping when absent.
    The known-key list names each such path.
  - A deeper key no template entry holds stays unknown and is refused; per-name maps of scalars or lists keep their dot-free rule.
- `configengine`'s `ValidateLocal` and `ValidateMachine` judge a unit `P.<name>.<rest>` whose name the template lacks against the first template entry at `P` holding `<rest>`, for existence and shape.
  Without this, the per-name declaration would refuse a profile the template lacks in a local or run-settings document, which the open-map declaration allowed.
- `batcher`'s loader records, for each profile, the highest layer file holding `profiles.<name>`.
  The retired-key refusal names that file, since a profile now merges from several layers and the top-level `profiles` origin can name a file holding no part of it.
- `configsync`'s `formerTemplateValues` entry for batcher shrinks to the one `cautious` leaf that was tuned, as a per-name map compares each leaf alone.
- Two YAML marshalling fixes that the per-name path exposed: the fill takes block style for a flow mapping it appends block entries to, and reconcile lifts a key's line comment over an empty collection onto the value, since either marshalled as invalid YAML.
- `doc.go` of `batcher`, `configengine` and `yamlengine` describe the new behaviour.

## Design notes

- A per-name map holds one kind of entry, so its first template entry decides scalar, list or mapping.
- Only a path some template entry holds is opened, and the category of `P` still decides in the local and machine checks, so per-name maps of a guard category, such as lenses' and gate's, stay closed to both layers.
  Behaviour of per-name maps for lenses and gate is otherwise unchanged.
- A hub `batcher.yaml` can no longer hide a template profile, since no layer deletes a key from the one below; `active:` is the only selection.
- A retired key in a lower layer's part of a split profile is attributed to the highest layer holding that profile.
- The registry fingerprint covers per-name maps, so hubs reconcile through `hubreconcile.Ensure` with no new code.
  Reconcile drops a profile from a standalone `batcher.yaml` once it equals the template, and the load resolves it from there.

## To check by hand

Not run here:

- In a hub, `lyx config batcher --layer default --set profiles.cautious.budget=300000` writes only that leaf, and `--resolved` keeps the other `cautious` keys.
- `--set profiles.wide.nope=1` is refused as unknown.
- The hub's own `batcher.yaml` copies of `cautious` and `identity` live on Bolt and are the operator's to remove; reconcile drops them once they equal the template.
