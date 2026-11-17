```yaml
lens: fix-code-correctness
outcome: reported
```

Lens: fix-code-correctness (run each changed function on a concrete input, check callers, check each added test fails with the defect in place).

**Findings**

1. **MEDIUM — F7 test can pass with the defect in place.**
   - Changed text: new row `APostToAnotherMachineStartsNoHubWorkerAtTheSchedulersStartOrOnThePost` in `internal/creelengine/machinemail_test.go`.
   - Evidence: the row asserts `hubs.Spawns(zzHub) == 0` right after `e.clock.AwaitWaitsWhere(1, isHourWait)`.
     - If a `wakeOutboundLocked` call were added to `postMachine`, the per-hub worker would start asynchronously.
     - It would connect, fail, and only then call `Spawn`. The fake counts the call even with no spawn function set (`creelfake/hubs.go:165-173`).
     - The hour-loop wait the test waits on is independent of that worker, so the assertion can run before the spawn happens.
   - The test therefore passes when it should fail, which is what the review's F7 asked it to catch. To make it deterministic, wait for the worker's own probe wait, such as `e.probeWaits(1)` when a worker is expected, then assert. A better option is to assert the absence of a per-hub worker through a signal instead of a spawn count.

**Checked and found correct**

- **F1:**
  - `replyAcrossMachinesLocked` now parses `original.From` with `ParseMachineAddress`. The sender was validated at `DeliverMachine`, so an unreachable parse error is just returned.
  - `noticeReturnedLocked` now takes `receivingMachine`, and both callers pass it: `entry.ReceivingMachine` for machine mail and `""` for per-hub mail.
  - The `agentname` and `strings` imports are consistent, and no separator split remains outside `internal/agentname`.
- **F2:**
  - The new `readChannelPass` returns `again` only for an operator command that moved the operator cursor while a message is held.
  - Reruns are bounded by `operators + 1`, and each rerun re-reads the entry and history from the persisted cursor. The inner `err` shadow is harmless because every return is explicit.
  - The reworked approve/revoke/plain row fails without the rerun: the held message is never delivered because the revoke runs first.
- **F3:**
  - `send` settles accepted posts first and re-reads the post record after `TakeMachine`. A settle that is lost and then answered by the next send no longer reposts, since the fake's `SettleMachineDelivered` drops the pending message.
  - The new test fails without the top-of-send settle: the second `SettleDelivered` call never happens and the `accepted` entry stays.
  - Cosmetic: when a settle fails and the message is still taken, the failing settle runs twice and its error is joined twice.
- **F4:**
  - The new `MachineUnsettled` case matches the constant.
  - The strengthened `notices[1]` assertion fails against the old default text.
- **F5:**
  - `ensureBridge` caches `bridgeHold{err}` for the not-listed error and returns `last.err` within the ceiling.
  - The mutex is held through the `switch` and released by the defer, so no lock leak.
  - The new row fails without the cache: a third send would call `CreateChannel` again, the failure queue holds only two entries, so the third call would succeed and the assertions on `creates` and `finds` would break.
- **F6, F8:** the `MaxNameLength` rename and the `MachineEnvelopeLead` use compile, `go vet` is clean and the gate is green. The `//testtiming:keep` line was reformatted by gofmt with a preceding `//` line, which is fine.

**Further tool calls**
1. `grep` over `internal/creelengine/outbound*.go`, `scheduler*.go` and `internal/testkit/creelfake/*.go` for spawn behaviour. I needed it to see when `Spawn` is counted, because the context did not show the fake's spawn timing.
