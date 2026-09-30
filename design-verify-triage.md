A red plan verify halts the whole run at Webster, whatever the cause.
Run `ide-hub-workspace` blocked twice on the flaky `TestRemove_AddSucceedsAfterWarpBranchDeleted` (see `fabric-readd-weft-push`), in a package the task did not break; the verify was green when rerun by hand.

Wanted, when the integration verify fails:
1. Rerun the failing tests once. Green on rerun: continue, and record the test as flaky in the run's report and friction.
2. Still red: run the same tests at the parent commit. Red there too: continue with a pre-existing-failure warning.
3. Halt only when a test fails on the task head and passes on the parent (a real regression).

Also: the integration report (`_lyx/webster/reports/integration.yaml`, `contracts/stencils/webster/webster-template-integration.md`) records only `status: OK | FAILED`; it must carry the failing test names and the tail of their output, so neither the driver nor the hub has to dig the fork transcript.