# Config layers set one profile key and merge profile maps by name

`config-profile-map-layers` · Task · bug, config, webster · running · Darn-Write

Two faults in how config layers handle a per-name map such as `batcher.yaml`'s `profiles:`.

## 1. `--set` of a nested key fails

Imported from [issue #833](https://github.com/Knatte18/loomyard/issues/833).

`lyx config batcher --layer default --set profiles.cautious.budget=300000` is refused with "cannot unmarshal !!str `300000` into batcher.profile", though the refusal's own known-key list names `profiles.cautious.budget`.

Expected: a key the verb lists as known sets that one leaf, leaving the profile's other keys to the lower layers.

## 2. A hub profiles map hides the template's profiles

Imported from [issue #834](https://github.com/Knatte18/loomyard/issues/834).

The hub's `batcher.yaml` added one profile, `wide`; `lyx config batcher --resolved` then listed only `wide`, and the template's `cautious` and `identity` no longer resolved.
Workaround in this hub: `cautious` and `identity` are copied into the hub layer, which pins their values there.

Expected: a layer's profiles add to or override the template's by name.
Once fixed, the hub's copies of `cautious` and `identity` can be removed (note `batch-budget-300k-trial`).
