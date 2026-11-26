# A shedverbs loop integration test hangs to the 20-minute package timeout instead of failing

`loop-test-hangs-not-fails` · Note · bug, shed, gate

Depends on: [`config-keys-nested`](design-config-keys-nested.md)

## Seen (2026-11-26, Webster's verify gate of `config-keys-nested`)

`TestLoopIntegration_ArmingRefusalStopsAsBootstrap` in `internal/shedverbs` sat at 0 % CPU for over 17 minutes.
Its child `lym shed step self` never exited, so the test waited until the package's `-test.timeout=20m` killed it.
The gate slot and the run were held for those 20 minutes before anyone saw a failure.
Whether `config-keys-nested`'s change caused the stuck step is for that run to find; this note is about the wait.

## Idea

A loop integration test that spawns `shed step` waits on it with its own bound, well under the package timeout,
and on expiry kills the child and fails with what the step last wrote (its stderr and status file),
so a stuck step costs seconds and names itself.
Check the other `TestLoopIntegration_*` tests for the same unbounded wait.
