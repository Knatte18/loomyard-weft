---
round: 1
focus:
  - "internal/orchcli/cli.go: the task's runs wiring beside the parent's switch of LoadConfig to fabricengine.ConfigDirs(location); settle whether every config load the task added in orchcli, battencli and loomcli uses the layered dirs rather than the old AnchorPath form."
  - "internal/loomcli/checklist.go, bootstrap.go and the removed statusbuild.go against the parent's new looplock.go, overlay.go, reviewstatus.go and wiring.go; settle whether the task's stage/checklist view knows the parent's new Merge segment and merge-review rows, and whether the removal of statusbuild.go left a caller or a test that assumed it."
  - "internal/battenshed/rownotice.go, notice.go and deps.go against the parent's VerifyRows, ParentLock and Merge-In row changes in the Land and verify wiring; settle whether the task's row notices still match the renamed or added rows."
  - "internal/battencli/overview.go and the runs view in internal/orchcli/runs.go; settle whether the overview shows or mishandles a run held by the parent lock or in the merge segment, and whether the new segment color 'merge' in reedengine templates is consumed consistently with the stage display."
  - "internal/testkit/envkit/envkit.go FullEnv: both sides edited the fixtures; settle whether the task's new deps (seat sharing, window close, overview) are filled alongside the parent's new fields, and whether any test is now redundant or any merged behavior uncovered, including cmd/lyx/testdata/test-budget.yaml and daemonwakeups_test.go against the parent's new wait-loop entry in pattern/PATTERN-daemon-wakeups.md (does any wait loop the task added need listing there)."
  - "Parent pattern changes (PATTERN-config-layers, config-strictness, hub-wide-config, durable-vs-ephemeral-state, wait-mark-display-only, told-geometry); settle whether the task's new config reads, persisted state and wait displays follow them."
---

Seeded from the overlap list and the two ranges; no round has been judged.
