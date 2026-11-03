```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour: pinning test**

- Rename of a former recipe key in each profile and under `defaults`, keeping values and comments: `TestMigrateConfig` row 1 (`migrate_test.go`) pins it. It checks the lines, the `# operator note` comment, and the values after `Load`. `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`, the new "profiles present under the recipe's former name" row, pins it again through reconcile.
- Former block beside a current one is dropped: `TestMigrateConfig` row 2 pins it.
- No-op leaves the document unchanged: `TestMigrateConfig` rows 3 and 4 (no former name, empty document) pin it. The unchanged-bytes check only runs when `wantLines == nil`.
- Unparseable document returns `ErrInvalid`: `TestMigrateConfig` row 5 pins it.
- Root that is not a mapping returns `ErrInvalid` (`migrate.go` non-mapping branch): none. This is a trust-boundary input, since the operator edits the file.
- `LackingTemplateBlocks` (starting profile, defaults, profile the template lacks, empty, unparseable, template itself): `TestLackingTemplateBlocks` pins all of these.
- A present seed-only file is written only when migrated (`Applied`), and `Migrated` and `Lacking` are carried in the result: the new profiles row in the configsync test pins the migrated case. The existing "broad" row, with `wantMigrated` and `wantLacking` both `[]string{}`, and the `Applied` assertion pin the no-write case.
- A migration error in `ReconcileHubWideAt` leaves the file untouched and returns a `FileError`: none. The configsync test has no unparseable `profiles` file row.
- `Lacking` set on a dry run too (the `Result.Lacking` doc says so): none. The test only reads `appliedResult`.
- Seed-only branch on a prime copy or legacy seed running `Migrate`: none. `profiles` is HubWide, so the prime is never read for it. The legacy-seeded path is untested, and `models` has no hooks. This is LOW.
- `lacking` key in the `lym config reconcile` envelope (`configcli.go:630`): none. Only the fixture in `configcli_integration_test.go` changed. It still pins that reconcile leaves retired module keys as written, now under `tapestry`.
- `hubreconcile/walk.go` Warn per module with lacking units: none.
- `Settings` refusing a block held only under the former name, with the reconcile as way forward: `TestFormerRecipeNameRefusal` pins it, and the battencli row 1 pins it through the Stuck wrap.
- `Default` returning an error for a former-name default, and `("", nil)` when no default exists: `TestFormerRecipeNameRefusal` pins both. The battencli row 2 pins the wrap in `prepareRunSettings`.
- `Default` signature change: `runprofile_test.go` pins it, with the call sites updated.
- `templateEntries` refactor: pinned by the existing `ConfigSpec` and template tests.
- `configreg` `Migrate` and `Lacking` wiring on the `profiles` row: pinned indirectly by the configsync profiles row, which goes through `configreg.Modules()`.

**Findings**

1. **MEDIUM, design**, `internal/configsync/configsync_test.go` (profiles rows near line 742): no row covers a `profiles.yaml` that fails to parse or migrate. `ReconcileHubWideAt` should return a `FileError` and write nothing for it. The focus directive names this "untouched with clear error" case. Evidence: the `migrate_test.go` unparseable row reaches only `MigrateConfig`, not the reconcile path (`configsync.go:1046-1052`). Fix: add a configsync row with `hubFile: "light: [\n"` asserting the error and that the file is unchanged.
2. **MEDIUM, design**, `internal/configcli/configcli.go:630`: nothing asserts the `lacking` key in the reconcile envelope. Evidence: the only `configcli` test change is the fixture edit at `configcli_integration_test.go:547`. Fix: add an assertion in an envelope test, with a profiles file lacking `sampler`, that `lacking` is listed and is absent when empty.
3. **LOW**, `internal/hubreconcile/walk.go:217`: the Warn for a seed-only file lacking units has no test. Fix: add a walk test with a logger capture, or accept it as log-only behaviour.
4. **LOW**, `internal/runprofile/migrate.go` non-mapping root branch: no row. Fix: add a `TestMigrateConfig` refusal row with `existing: "- a\n"`.
5. **LOW**, `internal/configsync/configsync_test.go`: dry-run `Lacking` and `Migrated` are not asserted, although `Result.Lacking` is documented as set on a dry run. Fix: assert the dry-run result in the new profiles row.

Tool calls: none; my context already held every file I needed.
