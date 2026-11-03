---
round: 2
focus:
  - "In internal/configsync/configsync.go, the seed-only branch of ReconcileHubWideAt runs Migrate and Lacking over a present profiles.yaml. Settle whether a file that fails to parse returns a FileError naming the profiles module before any write, leaving the file's bytes unchanged, and whether a test in internal/configsync/configsync_test.go reaches that path."
  - "In internal/configcli/configcli.go, reconcileEnvelope sets a lacking key when a result carries one. Settle whether a configcli test plants a profiles.yaml whose profile lacks a template block and asserts the envelope entry for profiles carries lacking, and that a module with no lacking units carries no such key."
  - "In internal/runprofile/migrate.go, MigrateConfig deletes a former block beside a current one. Settle whether the code, the MigrateConfig doc comment, the runprofile doc.go paragraph and the second row of TestMigrateConfig in internal/runprofile/migrate_test.go agree on what happens to a former block whose values differ from the current block."
  - "Settle whether any test added in the fix round duplicates the coverage of an existing test in the same package, and whether each new assertion pins behavior the diff adds."
---

Round 2 reviews the fixes for the round-1 test gaps and the former-block rule; the rest of the diff is settled.
