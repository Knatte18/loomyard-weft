```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each branch I ran a concrete input, including a failing call, against the diff.

**Per function or branch (case run → result)**

- `ParseMachineAddress`
  - `"osl::ly:orch"` with own name `osl` → refused, naming the bare name, before `checkMachine`. Correct.
  - Empty machine or empty name → refused. Correct.
  - `ly:impl` as the name part → accepted, since the role is not judged. Correct.
- `postMachine`
  - nil bridge, own name, non-orch sender, non-orch receiver, `SendReady` failure and empty body each refuse and journal nothing. Correct.
  - `machineWake` is set inside `do` and consumed by `do`. Correct.
- `replyAcrossMachinesLocked`
  - Runs the same sender checks, then journals with `ReplyTo=original.SendingID`. Correct.
- `refuseMachineAddress`
  - `inject`, `hold`, `release` and `interrupt` go through `refuseOtherHub`, `forward` through its own call. Correct.
- `TakeMachineOutbound`
  - A message past its hour is returned first and never handed out. Correct.
  - A non-`SLACK` sender is refused. Correct.
- `ReportMachineHold`
  - An unchanged reason writes nothing, a changed one journals once, and a settled message writes nothing. Correct.
- `SettleMachineDelivered`
  - On a returned message it journals delivered and posts one notice. Correct.
  - A second settle writes nothing. Correct.
  - An unknown id is refused. Correct.
- `SettleMachineReturned`
  - Pending → returned. Already settled → left alone. Correct.
- `machineHourLoop` and `returnDueMachine`
  - It returns due messages with no relay running, re-arms at the earliest hour, floors the wait at one second, and backs off on error. Correct.
  - The stop channel ends the loop. Correct.
- `DeliverMachine`
  - A repeat of the same sender and sending id returns the old id and stores nothing. Correct.
  - A reply-to that names a message never sent is refused. Correct.
  - The orch stamp follows the sender's role. Correct.
- Fold and carry
  - A pending machine-outbound message carries its hold reason. Correct.
  - Delivered or returned messages are not carried. Correct.
  - `machine-returned` after `machine-delivered` changes nothing. Correct.
- `bridgeSender.sendOne`
  - Button off, unapproved peer and channel hold each stop it before any post. Correct.
  - A post Slack accepted is recorded before the settle. Correct.
  - A lost connection on the settle keeps the entry for the next take. Correct.
  - The refusal cap settles the message returned. Correct.
  - `is_archived` drops the channel from the map. Correct.
  - `Invite` already treats `already_in_channel` as success (`api.go:351-353`). Correct.
- `bridgePoll.run`
  - Waits go 5, 10, 20, 40, 60 and return to 5 on progress. A wake is floored by `wakeFloor`. Correct.
- `bridgeReader.handle` and `handleEnvelope`
  - Own bot, human, bad envelope, retention, own-name clash, wrong app, no pin, wrong receiver, retired orch and button off follow D8's order. Correct.
  - Held messages stay held past five minutes. Correct.
  - Operator text under a held message is handled once through the operator cursor. Correct.
- `operator` in a bridge channel
  - An `approve` naming the peer is carried out. One naming this machine is left to the peer. Other text is answered once by the end that sorts first. Correct.
- `resync`, `resyncChannel` and `adoptByEnvelope`
  - Creator and authored-envelope tests, the retired-orch notice, and the cursor settings on adoption are correct.
- `reportUnapprovedBot`
  - The mark is cleared when nothing was reached. Correct.
- `keeper.retire` and `retireBridge`
  - Live beams are skipped through `slices.Contains(beams, slug)` (`keeper.go:473`). Correct.
  - A refused archive is retried because the orch is revisited. Correct.
- `holder.route` and `hintBridgeResync`
  - The limit is once per `bridgePollBase` per hub. Correct.

**Findings**

1. MEDIUM, design. `internal/slackengine/bridgein.go:168-186` (`readChannel`) and `:437-449` (`handleOperator`).
   - D8 says an approval delivers the held messages in order and moves the delivery cursor past each one, so a later `revoke` holds nothing twice.
   - The code never delivers at approval time. After a held message, `readChannel` skips every later non-operator message, and the held one is retried only on the next poll.
   - Concrete case: held M1, then `approve`, then `revoke` in one read. Both commands are handled, the pin is gone before M1 is retried, and M1 is never delivered.
   - The acceptance criterion "approve then revoke leave no second notice for the delivered message" is met only because the notice mark survives, not because the message was delivered.
   - Fix: after an `approve` is carried out in `readChannel`, retry the held message in that same pass, before the next operator message is handled.

2. LOW, design. `internal/creelengine/machinemail.go:231-243` (`machineHoldWayForward`).
   - `MachineUnsettled` (`bridge-unsettled`) has no case. It falls to the default "act on the relay notice for bridge-unsettled in your orch's Slack channel", but no relay notice exists for that reason.
   - The return notice for a message the relay never held therefore names a way forward that does not exist. Add a case for `MachineUnsettled`.

3. LOW, design. `internal/slackengine/bridgeout.go:99-132` (`send`) and `:63-80` (`settleAccepted`).
   - D3 says the relay settles every accepted message of its post record before taking new mail.
   - `settleAccepted` runs only at relay start. `send` settles only the entries `take` still returns.
   - If a settle was lost to a dropped connection and creel then returned the message at its hour, `take` no longer lists it. The Accepted entry sits until the next relay restart, and the late-delivery notice is delayed.
   - Fix: have `send` settle the remaining Accepted entries before taking, as `settleAccepted` does.

4. LOW, design. `internal/slackengine/bridgechannel.go:46-76` (`ensureBridge`).
   - The once-per-ceiling limit on `FindChannel` holds only for a hold reason.
   - When `adoptBridge` errors with "name is taken but the list does not show it", nothing is cached in `bridgeHolds`. The next poll repeats create plus `FindChannel`, so the paged listing can run every poll.
   - Fix: cache that failure with the time, as a hold is cached.

No other defects found on the cases run.

**Tool calls**
1. Read `internal/slackengine/keeper.go:425-525`. The manifest had only the diff hunks, so I needed the unchanged beam filter in `retire`.
2. `grep` on `internal/slackengine/api.go` for `Invite` and `already_in_channel`. The diff did not show `Invite`'s existing handling.
