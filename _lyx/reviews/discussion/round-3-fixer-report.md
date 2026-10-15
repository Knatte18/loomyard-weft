# Round 3 fixer report — discussion

## Changes

- **F1** (fixed as proposed): in D4's retired-key paragraph of `_lyx/discussion/decision-record.md`, deleted "and keeps an open-map flag".
  The dotted-prefix rule is now the only way a retired entry is admitted.
  It works because `yamlengine.LeafPaths` splits a local `claude_prompt_cache_ttl_roles` into per-role units once the key is outside `OpenMaps` (`internal/yamlengine/layers.go:370`).
- **F2** (fixed as proposed, the "not checked" branch): added a sentence to D4.
  It says a retired unit is judged by the retired-key rule alone, before the template-presence and shape checks (`internal/configengine/layers.go:169-176`), so its shape is not checked.
  A wrong-shaped value loads, is dropped from the merged document and is moved verbatim.
  `claude`'s local validation then refuses it naming `claude.local.yaml`, which blocks a run start, an open or a resume and never a stop or teardown.
  I picked this branch over keeping an old-shape check because the retired-key field maps only key to category and holds no shape, and because D4 says the move never changes a value.
  Added a matching clause to the acceptance row beginning "Before any reconcile".

## Disputed

None.

## Deferred

None.
