# Fixer-report, code review round 1

## Gate repair

The gate found the `internal/runprofile/doc.go` paragraph ending a sentence with a comma before the F6 sentence I added.
It now ends with a period, and the F6 sentence breaks at its semicolon, one clause per line.

## Changes

- **F1** (fixed as proposed): `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains a `wantFileError` field and a profiles row with `light: [\n`; the dry run and the apply both return a `*FileError` naming `profiles`, and the file's bytes are unchanged.
- **F2** (fixed as proposed): the `reconcile --layer default` subtest of `configcli_integration_test.go` plants a profile lacking its sampler block, asserts the `profiles` entry carries `lacking` `[light.sampler]` and the `loom` entry carries no `lacking` key, and restores the file in a `t.Cleanup`.
- **F3** (fixed as proposed): the same table now compares the dry-run result's `Migrated` and `Lacking` to `wantMigrated` and `wantLacking`; its `Applied` false was already asserted.
- **F4** (fixed as proposed): `TestMigrateConfig` gains a `- a\n` refusal row.
- **F5** (fixed as proposed): `TestEnsure_WarnsOfTemplateUnitsASeedOnlyConfigFileLacks` runs `Ensure` over a board whose `profiles.yaml` lacks its sampler block with the captured logger and asserts the Warn names the module and `light.sampler`, and that the file is left as written.
- **F6** (decision-class; carried out on the parent's recorded decision, creel #518): `MigrateConfig` deletes a former block beside a current one only when the two decode equal, and otherwise leaves the entry untouched with no line. `LackingTemplateBlocks` reports each differing pair as `<entry>.<from>: differs from <entry>.<to>`, which reaches `lacking` in the envelope and the walk's Warn. The second row of `TestMigrateConfig` now holds an equal pair, a new row pins the kept differing pair, and the configsync, `LackingTemplateBlocks` and walk tests pin the report. The `MigrateConfig` doc comment and the `runprofile` `doc.go` paragraph match.

## Self-check

- fix-code-correctness, MEDIUM: the Warn's wording and way forward, the `Result.Lacking` doc and the `lacking` envelope comment described only missing template units. Fixed as proposed; the Warn, its way forward, the doc and the comment now cover a pair to settle, and the walk test follows the Warn.
- fix-code-correctness, LOW: the planted `profiles.yaml` was not restored when the CLI call failed the test. Fixed as proposed; a `t.Cleanup` restores it.
- fix-consistency, MEDIUM: the Warn text and way forward. Duplicate of the first finding above; fixed.
- fix-consistency, MEDIUM: the `Result.Lacking` doc and the `lacking` comment in `configcli.go`. Duplicate; fixed.
- fix-consistency, LOW: the doc comments of `TestMigrateConfig` and `TestLackingTemplateBlocks`. Fixed as proposed.
- fix-consistency, LOW: the `migrate.go` file header. Fixed as proposed.
- fix-proposal, MEDIUM: the Warn's way forward for a differing pair. Duplicate; fixed with a way forward naming both kinds of entry, not a separate result field.
- fix-proposal, minor, no severity: the restore skipped on a `t.Fatalf`. Duplicate of the cleanup finding; fixed.
- fix-affected-set, LOW: `Result.Lacking`, `configcli/doc.go`, `hubreconcile/walk.go` and `hubreconcile/doc.go` wording. Fixed as proposed.
- fix-affected-set, LOW: the `configcli.go` `lacking` comment. Duplicate; fixed.

## Disputed

None.

## Deferred

None.
