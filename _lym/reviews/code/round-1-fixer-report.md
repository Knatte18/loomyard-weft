---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: >-
      The fix key offered two routes for the machine in the return notice; I took neither.
      noticeReturnedLocked takes the message's receiving machine, which creel already journals as ReceivingMachine,
      so no second parse is needed and the per-hub caller passes an empty machine.
      replyAcrossMachinesLocked uses agentname.ParseMachineAddress as proposed.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: >-
      The fix key kept a settle in the take loop's accepted branch.
      A self-check fork showed that branch repeats the settle the top of send just tried and doubles the call and the error on a lost connection,
      so the branch now only skips the entry, which still keeps it from being posted again.
  - id: F4
    disposition: fixed-differently
    reason: >-
      The fix key worded the notice as turning a slack button on.
      A self-check fork showed a message can only be posted while a button is on, so that advice can be a no-op;
      the wording asks the operator to check that this hub's slack relay runs and that a slack button of this hub is on.
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: >-
      The fix key counted hub starts only.
      A self-check fork showed a spawn is asynchronous and the count can read zero before a defect shows,
      so the row also asserts that the post journaled only its machine-outbound record, which any per-hub worker needs an outbound record to start.
  - id: F8
    disposition: fixed
---

# Fixer-report, round 1

## What changed

- F1: `replyAcrossMachinesLocked` parses `original.From` with `agentname.ParseMachineAddress`.
  `noticeReturnedLocked` takes the receiving machine, `machineHoldWayForward` takes the machine in place of the address,
  and no non-test file outside `internal/agentname` splits on the machine separator any more.
- F2: `readChannel` runs `readChannelPass` again from the persisted cursor after an operator command carried out under a held message,
  bounded by the operator's messages in the first read.
  The approve, revoke and plain-line row now reads all three in one pass and expects the held message delivered before the revoke stands.
  The row that unpins a machine moves its approve into a second read, since an approve in the first read now delivers at once.
- F3: `send` settles the accepted posts of the record before `TakeMachine` and joins that error into its failures.
  The take loop skips an entry the record still holds as accepted.
  A new `TestBridgeSend` row loses a settle, empties the take and expects the next send to settle and drop the entry.
- F4: `machineHoldWayForward` words `bridge-unsettled`; the `machinemail_test.go` row asserts the wording.
- F5: `ensureBridge` caches the not-listed error in `bridgeHold` and returns it until `bridgePollCeiling` passes.
  The row sits in `bridgeout_test.go` and not `keeper_test.go`, since it drives the sender and needs no new export from the keeper.
- F6: `machinename.MaxNameLength` and `creelengine.MachineEnvelopeLead` are exported and used in `crossmachine.go`;
  `bridgeEnvelopePrefix` is defined as `creelengine.MachineEnvelopeLead`.
- F7: a `TestStore_MachineOutbound` row posts to another machine under a started scheduler and asserts no hub start and no per-hub record.
- F8: `TestSlackMachine` carries the `//testtiming:keep` line; gofmt added a blank `//` line above it.

A gate repair then broke three comment lines of the round, in `bridgechannel.go`, `bridgein.go` and `bridgeout.go`, at sentence boundaries.
No code changed in it.

Commits are grouped by file set, not one per finding, since F1, F4, F6 and F7 share files and so do F3 and F5.

## Self-check

- fix-code-correctness, MEDIUM, F7 row can pass with the defect in place: fixed differently, as in the F7 entry.
- fix-consistency, LOW, `doc.go` says the notice names what clears `bridge-unsettled` and not what it is: no change, the sentence holds and the fork called it a missed chance only.
- fix-consistency, LOW, `relay.go` still settles accepted posts at start: no change, the start call reports under its own `bridge-settle` health key and the review asked for no removal.
- fix-proposal, LOW, F4 advice can be a no-op: fixed differently, as in the F4 entry.
- fix-proposal, LOW, F7 row can read zero spawns early: fixed differently, as in the F7 entry; the duplicate of the correctness lens's finding.
- fix-proposal, LOW, F2 reruns for an operator message that is no carried-out command: no change, it costs reads only, is bounded and is no worse than before the fix; the rubric excludes an improvement the review never asked for.
- fix-proposal, LOW, F3 failed settle runs twice in one send: fixed differently, as in the F3 entry; the correctness lens's cosmetic note is the same.
- fix-proposal, LOW, F5 cached error hides a freed name for up to a ceiling: no change, the fork rated it acceptable and a hold reason already waits the same ceiling.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
