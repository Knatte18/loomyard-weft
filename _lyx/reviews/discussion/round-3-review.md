---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:87-89
    summary: D4's retired-key field keeps an open-map flag that no stated rule consumes, since entry admission is by dotted-prefix match and the retired keys leave OpenMaps
    fix: In D4's retired-key paragraph, either delete the clause "and keeps an open-map flag" (the dotted-prefix rule in the next sentence already admits each `claude_prompt_cache_ttl_roles.<role>` entry), or state what the flag does, for example that its paths are passed to `yamlengine.LeafPaths` as open maps so the whole key is one unit and the prefix wording can be dropped. Keep exactly one of the two mechanisms.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:86-91
    summary: D4's retired-key admission skips the template shape check ValidateLocal applies today, and the Decision does not say what a wrong-shaped retired TTL value lets through or what bounds it
    fix: In D4's retired-key paragraph add one sentence on shape. State either that a retired key's value shape is still checked against the old template shape (scalar for `claude_prompt_cache_ttl`, mapping for `claude_prompt_cache_ttl_roles`), or that it is not checked, is moved verbatim, and is then refused by `claude`'s own local validation naming `claude.local.yaml`, which blocks a run start, an open or a resume and never a stop or teardown. Add a matching clause to the acceptance row beginning "Before any reconcile".
---

### [LOW] Retired-key open-map flag has no consumer

**Location:** `_lyx/discussion/decision-record.md:87-89`
**Issue:** D4 says the retired-key field "keeps an open-map flag", and in the same paragraph that the retired keys "leave `Categories` and `OpenMaps`".
`yamlengine.LeafPaths` (`internal/yamlengine/layers.go:370`) treats a key as one unit only at a path in the `openMaps` argument, and `ValidateLocal` calls it with `spec.OpenMaps`.
With the retired key outside `OpenMaps`, a local `claude_prompt_cache_ttl_roles` splits into `claude_prompt_cache_ttl_roles.<role>` units, and the dotted-prefix rule D4 states next admits each of them.
That makes the flag redundant, or it implies a second mechanism the Decision does not describe.
A plan writer cannot tell whether to build the flag or ignore it.
The focus question on where the open-map status lives is otherwise answered: D1 declares it for `claude.yaml`'s key, and D4 declares the retired entry and keeps `Categories` and `OpenMaps` to template paths, which satisfies `TestModules_EveryTemplateKeyClassified`'s "names no template path" check.

### [LOW] Retired-key admission leaves the shape bound unstated

**Location:** `_lyx/discussion/decision-record.md:86-91`
**Issue:** `ValidateLocal` today refuses a unit absent from the template (`internal/configengine/layers.go:169`) and a unit whose shape differs from the template's (`:172-176`) before it looks at the category.
D4 adds an admission path for retired keys: a `CategoryNone` retired key admits the equal unit and its dotted descendants.
D4 bounds that path by layer and category (the TTL keys at the local layer, Executable and Guard keys still refused), which the acceptance row pins.
It does not say whether the shape check survives for a retired key.
If it does not, a local `claude_prompt_cache_ttl: {a: 1}` loads, is dropped from the merged document, and is moved verbatim into `claude.local.yaml`, where `claude`'s own validation refuses it.
That outcome is acceptable, because the refusal falls on run start and never on stop or teardown, but the rubric's attack-surface question asks the Decision to state what the path lets through and what bounds it.
