---
round: 1
focus:
  - "internal/yamlengine/reconcile.go: check that the per-name drop compares decoded values (block vs quoted scalar, flow vs block list) and never drops or overwrites a name whose value differs, or a name the template lacks; settle whether key moves and `Migrate` hooks still write what they move."
  - "yamlengine.SetValues and the base-layer `--set`: the board entry names only reconcile, so settle whether the extra drop in `SetValues` is needed for the goal and leaves the loaded value unchanged, including a name a pair just set to its template value."
  - "Sparse layers (`yamlengine.Prune`, worktree and local layers): settle that no path drops template-equal names there, and that `configsync` calls the drop only at the two base-layer reconcile sites."
  - "Absent or null per-name map handling: settle that it writes an empty mapping, reports `P` added, and that the load's template fill still resolves every template lens and fan afterward."
  - "configengine's changed 'filled missing keys' Info line: settle that it drops only keys under a declared per-name map and still logs a filled per-name map path."
  - "Docs in the same diff: check the `lenses.yaml` header in internal/lenses/template.yaml, internal/lenses/doc.go, yamlengine and configengine doc.go, docs/overview.md, and PATTERN.md if a new invariant was added, against what the code now does."
  - "Summary claims against the diff, especially `Dropped` on dry runs, `--apply` rewriting on non-empty `Dropped`, the `dropped` output in `lyx config reconcile`, and the hub reconcile log."
  - "Test economy: check new `TestReconcile`, `TestSetValues`, `TestSet`, configsync, configcli and hubreconcile rows for overlap with each other or with existing tests, and for any added behavior left uncovered."
---

Seed focus from the board entry and the change description; no diff reading yet.
