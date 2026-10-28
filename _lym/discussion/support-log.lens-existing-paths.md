```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every existing code path that reads, writes or resets the state D1 to D7 change.

**Findings**

- **MEDIUM, D6/D5 (possible silent undo by reconcile).** D6's value may be stripped from the pair's local layer without notice, and the record doesn't say.
  - `internal/configsync/configsync.go` `formerLoomValues` records `code_reviewer_fan` with a former template value of `""` (it does the same for every `*_reviewer_fan` and `*_fixer_fan`).
  - D6 writes exactly `""` into the pair's local layer through Seed-Child.
  - `ReconcileAll` reconciles "a worktree's sparse and local layers", and line 658 passes `formerValuesDoc` into reconciliation.
  - If a former-template-equal value is dropped as a stale template fill, the next reconcile of the pair (`lym fabric reconcile`, or `hubreconcile.Ensure` after a binary change) removes `code_reviewer_fan: ""`. The pair then falls back to `["review-code", ""]`, and the `light`/`docs` code review runs the fan again.
  - I could not confirm whether the former-value strip covers the local layer or only base files. The record should settle this fact: either D6 survives local-layer reconcile, or the former value for `code_reviewer_fan` is dropped from `formerLoomValues` or kept out of the local layer.
- **LOW, D5 (doc and comment surfaces D7 misses).** Other prose still describes the fan keys as closed to the local layer:
  - `internal/loomengine/config.go`'s file comment, line 8.
  - The `ConfigSpec` doc comment ("Only the keys that can switch off or hollow a review ... are barred").
  - `internal/loomengine/template.yaml`'s `code_reviewer_fan` comment.
  - `internal/configcli/doc.go` line 56, which describes the `*_reviewer_fan` keys.

  Each still reads right until it states closure. D7 names only `docs/overview.md`.
- **LOW, D1–D3 (in-flight runs across the deploy).**
  - A run in the middle of Discussion-Review or Plan-Review when the binary changes resolves round N+1's fan with the new lens names.
  - The Bouncer judge only asks for `exclude_lenses` when round N+1 resolves to the same fan as round N. The fan name is the same, but its lens list changed (`goal-scope`/`board-fit` became `board-scope`; `way-forward` became `repo-invariants`).
  - The judge may therefore be asked to exclude a lens the next round no longer runs, or leave out the new one.
  - The effect is harmless: one extra or missing exclude for one round. The record is silent on runs in flight.

**Paths traced**

- D5: `configengine.LocalDenied`, which `configreg.withLocalBounds` derives from `Categories`, follows the reclassification. Survives.
- D5: `configreg.checkLocalDocument` (`ValidateLocal`, then `loomengine.ValidateLocalValues`, then `sheddriver.CheckLocalOverride` and `reviewbudget.CheckLocal`) reads no fan key beyond the category. Survives.
- D5: the `configreg/keymove.go` moves from darn `darn_reviewer_fan`/`review_fan` to `loom.code_reviewer_fan` now land a moved key in a local layer that admits it. Survives, and is more permissive than before.
- D5/D6: `loomengine.Config.Validate` resolves `code_reviewer_fan` through `lenses.ValidateFanKeys`, which skips `""` entries. Survives.
- D6: the `darn-recipe.yaml` and `pawl-recipe.yaml` Bouncer and BurlerReview rows read `reviewer_fan_setting: loom.code_reviewer_fan` per round from loaded config, so the local-layer value is read correctly and the round runs solo. Survives.
- D6: `ReconcileAll`'s local-layer reconcile with former values. May break (finding 1).
- D6: Seed-Child's rewrite of the local layer to profile plus run settings. Survives, since it writes the D6 value itself.
- D1–D4: the lenses reconcile drops hub entries equal to the template. The hub's `lenses.yaml` tunes only `docs-consistency`. Survives.
- D1–D3: `ResolveFan` at Bouncer and BurlerReview construction, for a hub fan naming a deleted lens. Breaks, but the record names this under Open risks.
- D1–D3: Bouncer `exclude_lenses` across the deploy. Minor (finding 3).

**Tool calls**

1. Grepped `code_reviewer_fan`/`CodeReviewerFan` consumers and `keymove.go`: the context lacked who reads the key besides `config.go`.
2. Grepped `ReviewerFan` and `ValidateLocalValues`: the context lacked the local-value checks.
3. Grepped the other consumers (`configsync`, `shedrecipe`) and `ValidateLocalValues`' body: the context lacked them.
4. Grepped the recipes for `reviewer_fan_setting`/`cluster_excludes`: the context lacked how the Code-Review rows read the fan.
5. Read `configsync`'s `formerLoomValues` and `sheddriver.CheckLocalOverride`: the context lacked what reconcile does with `""`.
6. Read the `configsync.go` header and its local-layer use sites: the context lacked whether reconcile touches local files.
