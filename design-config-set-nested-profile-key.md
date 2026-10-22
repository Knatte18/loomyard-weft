# config --set of a nested profile key fails: writes the scalar as the whole profile

`config-set-nested-profile-key` · Note · bug, config, webster

Imported from [issue #833](https://github.com/Knatte18/loomyard/issues/833).

Seen (2026-10-22): `lyx config batcher --layer default --set profiles.cautious.budget=300000` is refused with "cannot unmarshal !!str `300000` into batcher.profile", though the refusal's own known-key list names `profiles.cautious.budget`. Same for 200000 once the hub layer holds a `profiles.cautious` map.

Workaround: set the whole profile as a flow map, `--set 'profiles.cautious={batchifier: cost, ..., budget: 300000, weights: {...}}'`, which pins every key of the profile in the hub layer.

Expected: a key the verb lists as known sets that one leaf, leaving the profile's other keys to the lower layers. There is also no verb to remove a key from a layer, so a pinned profile stays until the interactive editor removes it.