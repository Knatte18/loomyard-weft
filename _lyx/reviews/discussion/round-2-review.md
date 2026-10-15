---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4, retired-key paragraph)
    summary: D4's retired keys keep their old local-layer categories and admit the open map claude_prompt_cache_ttl_roles, but the Decision never says where that classification and open-map membership live, and the two places it could live today are closed to it
    fix: In D4's retired-key paragraph, after the sentence that introduces the new configengine.Spec field, state the field's shape and its matching rule, for example - The field maps each retired dotted key to the category it held, and keeps an open-map flag; ValidateLocal admits a local unit when a CategoryNone retired key equals it or is its dotted prefix, so each claude_prompt_cache_ttl_roles.<role> entry is admitted, and refuses a unit covered by a retired key of any other category; the retired keys leave Categories and OpenMaps, which keep naming only template paths. Add to the acceptance row that begins Before any reconcile - a local claude_prompt_cache_ttl_roles holding two roles loads, and a local claude or claude_deny_agent_tool is still refused naming the key.
---

### [MEDIUM] D4's retired keys have no stated home for their category and open-map shape

**Location:** `_lyx/discussion/decision-record.md`, D4, the paragraph beginning "`shuttle`'s `ConfigSpec` declares the five source keys as retired keys".

**Issue:** D4 says a retired key "is admitted only at the layers and categories it was admitted at before: the TTL keys at the local layer, while the old Executable and Guard keys stay refused there", and that `ValidateLocal` "admits a retired key a local file holds".
That classification has nowhere to live under today's code, and D4 doesn't give it one:

- `TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go:96-99`) fails on any `Categories` entry that names no template path.
  Once D1 removes the five keys from `shuttle.yaml`'s template, they cannot stay in `shuttle`'s `Categories`, and the test's `wantLocalDenied["shuttle"]` row changes with them.
- `claude_prompt_cache_ttl_roles` is in `shuttle`'s `OpenMaps` today (`internal/shuttleengine/config.go:57-58`).
  `yamlengine.LeafPaths` (`internal/yamlengine/layers.go:368-370`) treats a key as one unit only at a declared open-map path.
  If the open map leaves `shuttle`'s spec along with the template key, a local file holding the map yields units `claude_prompt_cache_ttl_roles.<role>`.
  A retired-key check by exact name then misses them, and `ValidateLocal` refuses them with "key … is not in the shuttle template" (`internal/configengine/layers.go:168-169`).

So a plan writer can implement D4 as written and still reproduce the failure D4 exists to prevent (round 1's F1): a run-settings TTL map in an unreconciled `shuttle.local.yaml` makes every `shuttle.yaml` load refuse, stop and teardown included.
The opposite reading, where admission ignores category, lets a local `claude:` binary path through to a load, which is a guard the local layer holds today.
Either way the bound D4 states depends on a mechanism it leaves unspecified.
`Plan-Write` doesn't read `support-log.md`, so the shape belongs in the Decision.

## Rejected

- Focus 1, how the loader treats the moved TTL keys before reconcile: D4 settles it.
  The retired keys are admitted by `ValidateLocal` and dropped from the merged document, so `shuttle.yaml` loads with the key absent.
  Stop and strand removal load no engine config (D2), and an acceptance row pins the pre-reconcile pair.
  D4 states the admission's bound: only the TTL keys, at the local layer.
  The one gap is the mechanism, filed as F1.
- Focus 2, sparse-layer collisions: D4 settles it.
  The template-default overwrite is limited to base layers.
  At a sparse layer any target value is kept as a kept-target collision.
  Moves, kept-target collisions and template-default overwrites are all reported, and the acceptance row covers the worktree case.
  I also checked a hub worktree without the sparse marker (`configsync.pendingMigration`), whose full `shuttle.yaml` holds template-default `claude_*` values.
  Moved into a sparse worktree `claude.yaml`, those values shadow the repo default exactly as the old full-file keys shadowed it, and the strip compares them as it did before, so this is not a regression.
- Focus 3, D8's compile-error guarantee for a nil or empty scrub set: not a defect.
  D8 says an empty set still compiles and has `reedengine.New` panic on one as a programmer error; a nil set is empty.
  An acceptance row pins the failure.
  The constructor sites outside the CLIs (`internal/hubgeom/spawnorder.go:110`, `internal/pairteardown/teardown.go:366`) can import `shuttleengines` without an import cycle: `go list -deps ./internal/shuttleengines` holds neither package.
- `configreg` importing `shuttleengines` (D2): no cycle, since `shuttleengines`' dependency closure holds neither `configreg`, `configsync` nor `hubreconcile`.
- `shellguard` importing `gateslot`, whose closure reaches `configengine`, `lock` and `proc`: D10 names that departure from `PATTERN-leaf-packages` and gives the reason, so it isn't an unflagged violation.
