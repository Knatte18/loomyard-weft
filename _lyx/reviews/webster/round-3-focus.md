---
round: 3
focus:
  - "internal/loomcli/smoke_test.go, subtest 'failure before the first persist leaves a non-empty driver log' of TestSmokeRunStandalone: settle whether its driver-log assertion matches the error the poisoned status file (poisonStatusFile sets state to 'poisoned') provokes in the shedengine read gate, which reports an invalid state."
  - "internal/loomcli/smoke_test.go, subtest 'died driver proceeds to the success envelope and logs why' of TestSmokeBootstrapLifecycle: settle whether the driver-log read can still run before the spawned driver has written it, given that awaitRunLock in internal/loomcli/bootstrap.go returns halted on its first poll for any status whose state is not running, and whether the polling added there is bounded and deterministic."
  - "Publish failure: settle whether both failing tests, TestSmokeRunStandalone and TestSmokeBootstrapLifecycle in internal/loomcli, now pass against the committed code, checking the branch commits after the rejected head for any other test that still assumes the old unknown-field rig for poisoning status files."
---

Focus targets the two Publish-failure tests; the rest of the range is reviewed as usual.
