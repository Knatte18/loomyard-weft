```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: per change, the set I found.**

1. **`Profiles.Default` now returns `(string, error)`.**
   - The only production caller is `battencli/runsettings.go:29`, which handles the error and wraps it in `ErrInvalidRunSettings`. That wrap routes it to Stuck through `battenshed/seamchild.go:143`.
   - Test callers in `runprofile_test.go` (lines 107, 110, 237, 240) and `migrate_test.go:160,166` are updated.
   - `battenshed/deps.go:329` describes the sentinel as "profile unknown, holds no block for the recipe, or a key does not validate". That is generic wording and not wrong. Covered.
2. **New `Settings` refusal for a former-name block.**
   - Reached through `prepareRunSettings`, which wraps it in `ErrInvalidRunSettings`. The `battencli` test rows pin it. Covered.
   - `PATTERN-refusal-way-forward` binds only the webster, shed and loom packages, so it does not bind `runprofile`. The message carries a `way forward:` clause anyway.
3. **New `Module.Lacking` hook and `Result.Lacking`.**
   - Consumers: `configcli.reconcileEnvelope` (`configcli.go:631`) and `hubreconcile/walk.go:217`. Covered.
   - The `hubreconcile` log line at `walk.go:385` shows only `Added`, `Removed` and `Migrated`, which is fine.
   - `ReconcileAll` and the per-worktree path (`configsync.go:518,592`) never read `Lacking`. This is intended, since only the board dir carries `profiles`.
4. **Seed-only `Migrate` now runs at the board dir.**
   - `configreg/runsettings.go:77` also calls `module.Migrate`, but only for run-settings modules, and `profiles` is not one. Not affected.
   - `models` is the other seed-only module and has no hooks. Its result is unchanged.
   - Docs updated: `runprofile/doc.go`, `configcli/doc.go`, `hubreconcile/doc.go`, `configengine` `doc.go` and `layers.go`, `configreg` field docs, and the template header.
   - A grep of non-test `.go`, `.md`, `.yaml` and `.tmpl` files for "never rewrite" and "seed-only" found no remaining stale claim. The `modelspec/template.yaml:3` "never rewrites" is still true for `models`.
   - `contracts/`, `plugins/` and the `pattern/` backgrounds mention none of this.
   - `pattern/PATTERN-config-layers.md` is untouched by the diff and does not contradict it. It binds `configengine` as the sole reader and the hub reconcile as the sole binary-change path, and `MigrateConfig` reads through neither.
5. **Fixture change in `configcli_integration_test.go`.** It still plants the profile and checks it. Covered.
6. **Declared PATTERN changes.** The directive declares none, so there is nothing to land.
7. **Commit messages.** There is no `Pattern-Departure:` trailer, and none is needed.

**Findings:** none. I found no member of the affected set that the diff leaves wrong.

**Tool calls:**
1. grep for "never rewrite" and "seed-only" wording in code and docs: checks for stale claims the diff did not touch.
2. grep for `.Default(`, `Defaults[` and `.Migrated` consumers: finds every caller of the changed signature and output.
3. grep for "holds no … block", `ErrInvalidRunSettings` and "lacking" in `contracts`, `plugins`, `battenshed`, `battencli` and `hubreconcile`: finds other readers or asserters of the changed messages.
