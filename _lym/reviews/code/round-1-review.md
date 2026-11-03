---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configsync/configsync_test.go:742
    summary: "No reconcile test row covers a profiles.yaml that fails to parse: the new error path in ReconcileHubWideAt for a present seed-only file is unpinned."
    fix: "In TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate in internal/configsync/configsync_test.go, add a profiles row with hubFile \"light: [\\n\" that asserts ReconcileHubWideAt returns a *FileError naming the profiles module and that the file's bytes on disk are unchanged after the apply. The row needs an error expectation field in the table, as the existing rows expect none."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configcli/configcli.go:630
    summary: "The new lacking key of the reconcile envelope has no test: no assertion reads it, listed or absent."
    fix: "In the configcli test that drives lym config reconcile --layer default over a board, plant a profiles.yaml whose starting profile lacks its sampler block and assert the envelope entry for profiles carries lacking holding that profile's sampler entry; assert a module with no lacking units carries no lacking key."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/configsync/configsync_test.go:756
    summary: "The new profiles row asserts Migrated, Lacking and Applied of the applied result only; the dry run's Lacking and Migrated, which Result.Lacking documents as set too, are not asserted."
    fix: "In the same profiles row, compare the dry-run result's Migrated and Lacking to wantMigrated and wantLacking, and assert its Applied is false."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/runprofile/migrate.go:46
    summary: "The branch of MigrateConfig that refuses a document whose root is not a mapping has no test row."
    fix: "In TestMigrateConfig in internal/runprofile/migrate_test.go, add a wantRefusal row with existing set to \"- a\\n\"."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubreconcile/walk.go:217
    summary: "The Warn the hub reconcile walk logs for a seed-only file lacking template units has no test."
    fix: "Add a case to the hubreconcile walk test that runs the walk over a board whose profiles.yaml lacks a template block with a captured logger, and asserts one Warn naming the module and the lacking units. If the walk tests have no logger capture, leave the Warn as log-only behaviour and drop this finding."
  - id: F6
    severity: LOW
    class: decision
    origin: handler
    location: internal/runprofile/migrate.go:62
    summary: "A former block beside a current one is deleted whatever it holds, so operator values that differ from the current block are discarded and survive only in the Bolt checkout's git history."
    fix: "Decide the rule. If the former block should survive when it differs, skip the rename and the delete for an entry whose former block is not equal to the current block, and emit no line for it; delete only an identical one. Then update the second row of TestMigrateConfig, the MigrateConfig doc comment and the runprofile doc.go paragraph to match. If dropping is intended, leave the code and record the choice in the change description."
---

### [MEDIUM] No test for a profiles.yaml that fails to parse during reconcile

**Location:** internal/configsync/configsync_test.go:742
**Issue:** Before this change a present seed-only file was never read, so a malformed `profiles.yaml` could not fail a reconcile. The seed-only branch of `ReconcileHubWideAt` now runs `Migrate` and `Lacking` over it and returns a `FileError` before any write (`configsync.go:980-986`, `1046-1052`). The focus directive asks to settle that such a file is left untouched with a clear error. The `migrate_test.go` unparseable row reaches only `MigrateConfig`, not the reconcile path. The code is correct by reading; no row would fail if it were broken.

### [MEDIUM] No test for the lacking key of the reconcile envelope

**Location:** internal/configcli/configcli.go:630
**Issue:** `reconcileEnvelope` now sets `lacking` when a result carries it, which is the operator-visible half of the goal's second Expected item. The only `configcli` test change is the fixture edit in `configcli_integration_test.go`, so a regression that drops the key would pass every test.

### [LOW] Dry-run Migrated and Lacking are not asserted

**Location:** internal/configsync/configsync_test.go:756
**Issue:** `Result.Lacking` is documented as set on a dry run too, and `lym config reconcile --layer default` without `--apply` is how an operator first sees it. The new row compares only the applied result.

### [LOW] The non-mapping root refusal of MigrateConfig is untested

**Location:** internal/runprofile/migrate.go:46
**Issue:** The operator edits this file, so a top-level list is a trust-boundary input. `TestMigrateConfig` covers an unparseable document and an empty one, not a parseable non-mapping root.

### [LOW] The walk's Warn for lacking units is untested

**Location:** internal/hubreconcile/walk.go:217
**Issue:** The Warn is behaviour the diff adds and no test reaches it. It is log output only, so the gap is minor.

### [LOW] A differing former block beside a current one is silently discarded

**Location:** internal/runprofile/migrate.go:62
**Issue:** The goal's first Expected item says the rename keeps the operator's values. When a profile holds both `loom` and `tapestry`, the code deletes `loom` without comparing the two, so values that differ are lost from the file. The current block is the one a run reads, so no active behaviour changes, and the drop is reported as a `removed, ... already set` line. The goal does not say what to do with such a pair, so this is a choice the author must make and the parent may already hold a view on it.

## Rejected

- Lens code-correctness F1, a malformed `profiles.yaml` now blocks the start verb's reconcile: the rule matches how every non-seed-only module treats a malformed file, the error is clear, and the missing test is F1 above.
- Lens code-correctness F2, `MigrateConfig` re-serializes the file at indent 2 and loses blank lines: the doc comment promises comments and other keys only, which hold, and the rewrite happens only when a former name is present. The drop of a differing former block is F6.
- Handler: the hub walk's Warn on every binary change for a block the operator removed on purpose: the goal licenses reporting a missing block, and the Warn's way forward says to leave it out on purpose.
- Handler: `mappingKeyIndex` in `migrate.go` repeats the helper in `internal/darnengine/config.go`: two call sites in different packages do not call for a shared helper under the abstraction rule.
- Lens code-affected-set returned no finding; the callers, docs, `pattern/` backgrounds and commit messages it checked agree with the diff, and no `Pattern-Departure:` trailer is needed.
- Summary claims, checked against the diff: the rename, the report-only `Lacking` hook, the seed-only `Migrate` run, the `Default` error, the Stuck routing through `ErrInvalidRunSettings` and the unchanged per-worktree path all hold. The three Expected items of the goal are met, and nothing beyond them is added without a basis in the goal other than the Warn rejected above.
