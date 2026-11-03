---
round: 1
focus:
  - "internal/runprofile/migrate.go: settle whether MigrateConfig renames both profile recipe blocks and `defaults` entries while keeping the operator's values, and whether the 'former block beside a current one is dropped' rule can discard operator data that differs from the current block."
  - "internal/configsync/configsync.go and internal/configreg/configreg.go: settle whether running Migrate over a present seed-only file writes only when something migrated, preserves the file header and other content, and leaves a file that fails to parse untouched with a clear error."
  - "internal/runprofile/runprofile.go Seed-Child path: settle whether the refusal fires only when the profile or default is held solely under the recipe's former name, and whether its message names `lym config reconcile --layer default --apply`."
  - "LackingTemplateBlocks and the Result.Lacking reporting in configcli and hubreconcile/walk.go: settle whether the three Expected items of the goal are all met (rename, lacking-block report for `sampler`, Seed-Child pointer) and whether anything beyond them is added without the goal licensing it."
  - "Tests (migrate_test.go, configsync_test.go, runsettings_test.go, configcli_integration_test.go): settle whether each new behavior is covered once, and whether the changed fixture in configcli_integration_test.go still pins what it did before."
  - "_lym/landing/summary.md against the diff, and the commit message claims of a9a641ac."
---

Decision-record-alone review: the measuring stick is the issue-29 board entry under `## Goal`.
