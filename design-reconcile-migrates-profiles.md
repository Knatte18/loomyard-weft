# config reconcile never migrates profiles.yaml, so a hub seeded before the loom-to-tapestry rename stops tapestry runs at Seed-Child

`reconcile-migrates-profiles` · Note · bug, reconcile, profiles, config

Imported from [issue #29](https://github.com/Knatte18/loomyard/issues/29).

## What happened

Hub NORCE-DrillingAndWells/Models. Its `_bolt/_lym/config/profiles.yaml` was seeded by an older build: recipe blocks keyed `loom` (and `defaults.loom`), no `tapestry` and no `sampler` blocks. Apart from those keys it was the untouched seed.

After the recipe rename, every tapestry run stopped at Seed-Child (seen on `commonlib-fluid-report-selection`, profile `light`), since the profile has no tapestry block.

`lym config reconcile --layer default --apply` from the prime reports `profiles` with `removed: null` and changes nothing, because the file's header says it is "seeded once when this file is absent; after that it is operator-owned and no reconcile rewrites it".
So no lym verb brings an existing hub's profiles in line with a renamed or added recipe. The fix was rewriting every profile by hand with `lym config profiles --set <name>='<json>'`.

## Expected

- A recipe rename in lym migrates the hub's profiles: `reconcile` (or the upgrade path) renames recipe keys it knows were renamed, `loom` -> `tapestry` in each profile and under `defaults`, keeping the operator's values.
- A recipe the template has and the hub's profiles lack (`sampler`) is reported by reconcile, or added from the template, rather than silently missing.
- Seed-Child, on a profile with no block for the task's recipe but a block under a known old name, names the reconcile or migration as its way forward.