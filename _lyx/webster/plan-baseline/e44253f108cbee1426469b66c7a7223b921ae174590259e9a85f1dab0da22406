# Card 4 — retired-config-keys

**Create:**
- `internal/configengine#retiredCovering`
- `internal/configengine#dropRetired`

**Edit:**
- `internal/configengine#Spec`
- `internal/configengine#ValidateLocal`
- `internal/configengine#load`
- `internal/configengine_test#TestValidateLocal`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine/doc.go`

**Intent:**
Give `configengine.Spec` retired keys, so a module whose keys moved to another module still loads an unreconciled layer that holds them (D4), which keeps every `shuttle.yaml` load, and so every stop and teardown, working across the deploy.

`Spec` gains the field `RetiredKeys map[string]Category`: each dotted key a module no longer reads, mapped to the category it held while it was a template key.
Retired keys stay out of `Categories` and `OpenMaps`, which keep naming only template paths.

`retiredCovering` returns the retired key equal to `path` or a dotted prefix of it, with its category, and whether one covers it; the longest such key wins.
`ValidateLocal` judges a local unit a retired key covers by that rule alone, before the template-presence and shape checks: a `CategoryNone` retired key admits the unit, so each `claude_prompt_cache_ttl_roles.<role>` unit of a retired open map passes, and a retired key of any other category refuses it with `key %q is %s and cannot be set in a local override`, marked `ErrInvalid` and naming the key, as a template key of that category is refused today.
A retired unit's value shape is never checked, so a wrong-shaped value such as `claude_prompt_cache_ttl: {a: 1}` passes local validation.
Bound: a retired key is admitted only at the layers and categories it was admitted at before, since the categories carry over; this admits nothing the template key did not.

`dropRetired` removes every top-level key a retired key names, and each dotted retired key's own entry, from a document, keeping every other key and comment.
`load` calls it on the stacked document right after `StackLayers` and before `yamlengine.FillMissing`, so a retired key held by any layer resolves as absent and no loader ever reads one.
Nothing else changes: `Layers` still validates a local file through `ValidateLocal`, and reconcile still prunes and, from card 8 on, moves a retired key, since `RetiredKeys` is read only by `ValidateLocal` and `load`.

Tests extend the existing tables:
- `TestValidateLocal` gains a spec with retired keys and rows: a retired `CategoryNone` scalar accepted; units under a retired `CategoryNone` map (two entries) accepted; a retired `CategoryNone` key holding a mapping where the template never held it accepted; a retired `CategoryGuard` key and a retired `CategoryExecutable` key each refused naming the key and marked `ErrInvalid`; and overlapping retired keys, a `CategoryGuard` map `old` with a `CategoryNone` entry `old.open`, admitting a unit under `old.open` and refusing one under `old.shut` naming `old`, so the longest covering key decides.
- `TestLoad_LayerStack` gains rows where a worktree layer and a local layer hold retired keys and the resolved document lacks them, every other key resolving as without them, and a row whose spec retires the dotted key `nested.gone` while the template keeps `nested.deep`: a worktree layer holding both resolves `nested.deep` and lacks `nested.gone`.

`configengine`'s `doc.go` describes retired keys.

**ImpactSummary:** One new `Spec` field read only by `ValidateLocal` and `load`; no module declares a retired key yet, so every existing load and local refusal is unchanged.
