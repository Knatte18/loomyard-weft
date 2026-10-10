---
round: 1
focus:
  - "Publish failure: `TestSmokeBootstrapLifecycle` subtests `died_driver_proceeds_to_the_success_envelope_and_logs_why` and `malformed_status_proceeds_to_the_success_envelope_and_logs_why` (`internal/loomcli/smoke_test.go`). The log shows `state: decode failed` refusing the bootstrap before the driver ran. Settle whether `currentProducer` in `internal/loomcli/start.go` is the only strict status read on the `lyx loom start` path before the spawn, or whether another site (Seed step, midmerge record read, resume own-leftover read) still returns `state.ErrDecode` as a refusal."
  - "Same failure: settle whether the `state.ErrDecode` fallback in `currentProducer` keeps the ordinary merge refusal over a parked mid-merge state, as cards 01, 03 and 06 of the plan require, and whether `start_midmerge_test.go` covers both the decode-failure and the parked-merge case."
  - "The merge-in commit 6235b4043 (`git diff 6235b4043^1 6235b4043`): look for parent-branch changes to the status file schema, `state.ReadJSONStrict` callers, or driver-log writing under `.lyx/loom/driver.log` that interact with the new midmerge record fields from plan card 01 and make the smoke test's rigged status files (unknown field, truncated JSON) fail earlier than before."
  - "Per-card mechanical check for each plan card 01-08: confirm each group's type-specific check from the plan spec ran against that group's own targets, and that the diff matches each card's `Produces`."
  - "Test economy: new tests in `internal/fabricengine` (config changes three-way, carried-over parent change row) and `internal/loomcli` that duplicate an existing test's coverage; name the covering test."
---

Seed focus for round 1, set from the Publish failure section, its log and the last commit `d2ad9fc55`, which already targets `currentProducer`.
