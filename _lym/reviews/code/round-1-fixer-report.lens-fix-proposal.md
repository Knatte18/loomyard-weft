```yaml
lens: fix-proposal
outcome: reported
```

The F8 mark matches the existing form, so there is no finding there.

Lens fix-proposal, findings fixed as proposed: F1, F2, F3, F4, F5, F6, F7, F8.

**F4 — LOW.** The `MachineUnsettled` case in `machineHoldWayForward` (`internal/creelengine/machinemail.go`).
- **Summary:** The advice to turn a slack button on can be a no-op, so the notice may misdiagnose.
- **Evidence:** `postMachine` calls `bridge.SendReady`, and the `MachineBridge` doc says it answers nil only when "it has a slack button that is on". A message can therefore only exist when a button was on at post time.
- **Case:** An unreported message comes back at its hour. The relay may have been down, crashed or not polling, or tokens may be missing. The button may also have been turned off after the post, but then the relay would report `button-off`, so that is not this reason. The notice tells the orch to turn on a button that is already on, with no step for checking that the relay is running.
- **Possible fix:** Word it as "ask the operator to check that this hub's relay runs".

**F7 — LOW.** The new row `APostToAnotherMachineStartsNoHubWorkerAtTheSchedulersStartOrOnThePost` in `internal/creelengine/machinemail_test.go`.
- **Summary:** The row may pass with the defect in place.
- **Evidence:**
  - It waits only on `e.clock.AwaitWaitsWhere(1, isHourWait)`, which signals the machine hour loop. It does not wait for the per-hub worker.
  - A defect such as a `wakeOutboundLocked` call in `postMachine` starts the hub worker asynchronously, so `hubs.Spawns(zzHub)` can still read 0 at the check.
  - Nothing synchronises with a possible spawn. The sibling reply row checks `Spawns` after `TakeMachineOutbound` and has the same weakness.
- **Possible fix:** Wait on a per-hub probe wait, such as `e.probeWaits(1)`, or fake a spawn signal before asserting, or have the row show it fails with the wake added.

**F2 — LOW.** `readChannelPass` and `readChannel` in `internal/slackengine/bridgein.go`.
- **Summary:** `commandDue` is true for any new operator message, not only a carried-out command.
- **Evidence:**
  - It is also true for `approve home`, which `handleOperator` leaves to the other relay, and for plain text answered "not delivered".
  - Each such message while a message is held costs a rerun: another `History` call plus the held message's envelope checks.
  - The bound of operators+1 keeps this finite. The operators count also includes already-handled operator messages, so the bound is looser than needed.
  - This costs reads only and does not change behaviour.

**F3 — LOW.** `send` in `internal/slackengine/bridgeout.go`.
- **Summary:** A failed settle is attempted twice in one send.
- **Evidence:**
  - When the top `settleAccepted` fails on a lost connection, the entry stays in the record.
  - `TakeMachine` then returns the still-pending message, so the loop branch settles it again in the same send.
  - That makes two calls to an unreachable creel and the same failure twice in the joined error.
  - The reviewer asked to keep the branch, and it is correct as a guard against reposting. The doubled call and error are a side effect.

**F5 — LOW.** `ensureBridge` in `internal/slackengine/bridgechannel.go`.
- **Summary:** A cached not-listed error also hides a freed name.
- **Evidence:**
  - Within `bridgePollCeiling`, `ensureBridge` returns `last.err` without calling Slack.
  - If the other channel is renamed or the listing catches up, the create that would now succeed waits up to a ceiling.
  - This matches the delay hold reasons already have, so I rate it acceptable and do not count it as a defect.

**F1, F6 and F8 — no findings.**
- **F1:** The parse in `replyAcrossMachinesLocked` takes `original.From`, which `DeliverMachine` validated with the same rule. `entry.ReceivingMachine` is set at every `machine-outbound` append, including the reply path (`original.SendingMachine`).
- **F6:** Exporting the constants breaks no caller.
- **F8:** The mark matches the existing form at `contracts/stencils/engineterms_test.go:255-256`.

Calls made:
1. `sed` of `contracts/stencils/engineterms_test.go` lines 250-258, plus `grep` for `testtiming:keep` outside tests: my context lacked a sample of how the directive is placed after a doc comment, which F8's `gofmt` blank `//` line depends on.
