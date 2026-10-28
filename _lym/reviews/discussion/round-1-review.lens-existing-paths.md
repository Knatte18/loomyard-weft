```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over decision-record.md and support-log.md for review-setup-tuning.

**Paths traced, per decision**

- D1, `internal/lenses/config_test.go:288–343`, the emphasis-line check: the procedure-lens exemption comes only from `review-*`/`fix-*` fans. Once `board-scope` leaves `review-discussion` it sits only in `twill-discussion` and `discussion`, so the test requires it to end with the emphasis line. **Breaks** D1's "Ends in its list" and the Constraint.
- D1, the template header rule at `internal/lenses/template.yaml:33–41`: "procedure lenses of the production fans (each review-* and fix-* fan)". `board-scope` is in no such fan, so the header's own rule classes it as an emphasis lens. **Breaks**, same defect as above.
- D1, the writer's route for a parent question (`loom-template-discussion.md:94–96`): the writer records it under Open risks, and the Discussion-Review perch raises it. This holds with `review-discussion` left without a board lens. **Survives.**
- D1/D3, a hub `lenses.yaml` that tunes deleted-lens prose or lists it in `remove:`: tuned prose is kept as an extra entry and reported; a `remove` name that matches no lens is skipped with a warning. **Survives.**
- D1/D3, a hub fan naming a deleted lens: `ValidateFanKeys` → `ResolveFan` "names undefined lens". Covered by Open risks. **Survives** as stated.
- D2, the writer-covers-reviewer check with `absorbedBy`: the acceptance criterion covers it. **Survives.**
- D3/D4, the emphasis exemption for `code-*`: these lenses stay in `review-code`, so they stay procedure lenses. **Survives.**
- D5, `PrepareRunSettings` → `module.Migrate` (loom `MigrateConfig`) → `checkLocalDocument`: `webster_reviewer_fan` is renamed onto `code_reviewer_fan` before the check, so it is now admitted too. D5 names only the darn keys. **Breaks** D5's route list (see F2).
- D5, the local-layer write through `configengine.Set` → `ValidateLocal`, and the load-time `ValidateLocal` at `layers.go:480`: both now admit the key. **Survives.**
- D5, reading the value: `SettingLookup` → `fanSetting` → Bouncer and BurlerReview. A wrong name is caught first by `LoadConfig` at `loomcli/wiring.go:378`, which loom, darn and pawl runs all wire through. **Survives.**
- D5, Seed-Child resume: it redoes the write from the current board entry (`battenshed/doc.go:103–109`). This matches "reach a pair only at Seed-Child". **Survives.**
- D6, the local file through `configsync.ReconcileAll` → `reconcileSparseFile(..., nil)`: `Prune` keeps the key because it is a template key, and no former-value strip runs. **Survives.**
- D6, the `hubreconcile` walk (`walk.go:285`) over every worktree: it runs `ReconcileAll` with Apply and Migrate, and the local file still takes the nil strip. **Survives.**
- D6, Seed-Child writes the block → `LoadConfig` reads `[""]` and `ValidateFanKeys` skips the empty entry → `fanSetting` turns `""` into nil fans, so the round runs solo. **Survives.**
- D6, the hub `profiles.yaml` and the `checkProfiles` write check: the old binary refuses the key until the deploy. Open risks orders the re-set after the deploy. **Survives.**

**Findings**

- F1. MEDIUM, class `design`, D1 and Constraints ("procedure lenses end in their own list").
  Once `board-scope` leaves `review-discussion`, it is in no `review-*`/`fix-*` fan.
  The template's lens rule and `TestTemplate`'s emphasis check both class it as an emphasis lens that must end with "Report anything else you notice too — emphasis, never exclusion."
  D1 instead has it end in its own list and "report only" contradictions, and neither the record nor the acceptance criteria say which rule gives way.
  Evidence: `internal/lenses/config_test.go:288–298, 336–343`; `internal/lenses/template.yaml:33–36`.
- F2. LOW, class `consistency`, D5 Bound and its acceptance criterion.
  The record lists the routes onto `code_reviewer_fan` as profile block, `run_settings`, local layer and the darn keys.
  Loom's own retired `webster_reviewer_fan` reaches the key too, because loom's Migrate runs before `checkLocalDocument`, and the record does not name it.
  Evidence: `internal/loomengine/config.go:1035` (`retiredPerchKeyRenames`); `internal/configreg/runsettings.go:72–84`.

Outside my lens: D7 names loom's `template.yaml` key comment and `internal/configcli/doc.go` as calling the fan keys closed, but grep finds no such text in either. That claim belongs to bound-breaker or the handler.

Tool calls:
1. Grep of the discussion writer stencil and the twill stencils for how a `decision`-class parent question is routed. My context lacked the writer's routing rule.
2. Read of `twill-writer-fork.md`'s head plus grep of `hubreconcile` for `ReconcileAll`/`LocalFile`. My context lacked the writer-fork report shape and whether `hubreconcile.Ensure` touches local files.
