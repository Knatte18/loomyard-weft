---
round: 1
focus:
  - "internal/shuttleengine (run.go, wait.go, engine.go, fakes_test.go, claudeengine.go): the parent reworked the launch line and waits; check the task's level-policy changes (Info once per state, no per-poll logging) still hold in the parent's new wait and run paths, and that the task's tests use the parent's changed fakes."
  - "internal/battenshed (innerrun.go, notice.go, publishretry.go, doc.go) and internal/battencli/wire.go: the task sends a landed notice on every landing; check the parent's changes to the notice, innerrun halted path and publish retry did not leave a landing path that sends no notice or sends it twice."
  - "internal/pairteardown/teardown.go with pattern/PATTERN-fabric-destruction-chokepoint.md: the parent changed the destruction-chokepoint pattern; check the task's archive-before-removal step still goes through the chokepoint the pattern requires."
  - "pattern/PATTERN-driver-choice-single-site.md and PATTERN.md: check the task's new call sites (watchdog respawn via reedengine.SpawnWatchdog, logger stderr discard under go test) follow the parent's changed or added invariants."
  - "contracts/stencils/orch/orch-template-role.md and contracts/stencils/claims_test.go: both sides edited the orch stencil; check the merged stencil still handles the landed notice and the parent's additions without contradiction, and that the claims test covers both."
  - "cmd/lyx/testdata/test-budget.yaml and the overlapping test files (innerrun_test.go, notice_test.go, wait_test.go, run_test.go): check the merged budget rows match the merged top-level test counts, and name any test the merge made redundant with another."
---

Seed focus drawn from the overlapping-file list and the parent's four commits; each entry names a place and a question, not a verdict.
