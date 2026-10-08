# Card 4 — reconcile-migration-hook

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configsync#Result`
- `internal/configsync#ReconcileAll`
- `internal/configcli#runReconcile`
- `internal/configcli#Command`
- `internal/configcli/doc.go`
- `internal/configsync/configsync_test.go`
- `internal/configcli/reconcile_integration_test.go`

**Uses:**
- `internal/batcher#MigrateConfig`

**Intent:** `lyx config reconcile` performs a module's declared migration before it reconciles the file against the template (D12), so a seeded `batcher.yaml` survives this task's retirement of `master_base`.
`configreg.Module` gains `Migrate func(existing []byte) ([]byte, []string, error)`, nil for every module but batcher, whose entry in `Modules` sets `batcher.MigrateConfig`.
`ReconcileAll` runs a non-nil `Migrate` over a present file's bytes first and reconciles the migrated bytes, so the migrated document is what is written under apply; an absent file is not migrated (it is seeded from the template).
A migration that changed something counts as a change, so `--apply` writes the file even when the reconcile itself adds and removes nothing.
`Result` gains `Migrated []string`, the hook's rewrites, set on a dry run too.
`runReconcile` adds a `migrated` key to a module's JSON entry only when it is non-empty, the way `migratedFrom` is reported.
A migration error fails the reconcile naming the module, as a reconcile error does today.
Bound: the hook rewrites only the keys its module declares retired and changes the operator's file only under `--apply`, after the dry run has listed each rewrite, which is reconcile's existing contract; the loader never runs it.
`configsync_test.go` extends `TestReconcileAll` with rows for the wiring only, since card 3's `MigrateConfig` table already pins what the rewrite keeps: a batcher file carrying `master_base`, whose dry run reports the rewrites in `Migrated` and writes nothing, and whose apply writes a file `batcher.Active` loads; and an absent batcher file, which is seeded from the template and not migrated.
The integration test asserts the `migrated` key in the envelope.
`internal/configcli/doc.go`'s reconcile paragraph and the `reconcile` command's long help in `Command` state that reconcile first runs a module's migration, that the dry run lists each rewrite, and the `migrated` key.
Other `configreg.Module` literals in tests compile unchanged since the field is optional.

**ImpactSummary:** Adds an optional hook field to configreg.Module and a Migrated result field; only batcher sets the hook, and fabric clone's reconcile picks the migration up through ReconcileAll unchanged.
