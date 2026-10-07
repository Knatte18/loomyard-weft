# Card 2 — Register the shuttle open map in configreg

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestLookup`

**Uses:**
- `internal/shuttleengine#ConfigOpenMaps`

**Intent:**
The `shuttle` entry in `Modules` gains `OpenMaps: shuttleengine.ConfigOpenMaps()`, as the `batcher` entry does.
Without it `lyx config reconcile` and `lyx config shuttle --set` treat `claude_prompt_cache_ttl_roles` as a closed map: reconcile prunes an operator's own entry such as `orch` and re-adds `driver` and `webster` to a map that left them out (D2).

`TestLookup` gains a `shuttle` row whose `wantOpenMaps` is `[]string{"claude_prompt_cache_ttl_roles"}`.

**ImpactSummary:** one registry entry gains an open-map declaration; configsync and configcli read it through the existing `Module.OpenMaps` path with no code change.

**Commit:** 2: configreg: declare shuttle's per-role TTL map an open map
