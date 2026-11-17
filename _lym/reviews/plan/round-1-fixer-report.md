---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: >-
      The proposed entry list misplaces one removal and misses four members.
      `GoClient.DeliverMachine` has no caller at card 10: only card 11's reader calls `Mailbox.DeliverMachine`, so its entry is removed by card 11, not card 10.
      Card 10's own `programMailbox.DeliverMachine` and `bridgePoll.wake` stay unreached until card 11's reader and wake loop call them, so card 10 adds entries for both and card 11 removes them.
      Card 2's `machineOutboundEntry.message` has no caller until card 3's `settledOutcomeLocked`, so card 2 adds an entry and card 3 removes it.
      Every other entry is as proposed.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

# Round 1 fixer report — plan

`lym loom validate-plan` passes on the plan as changed.

## Changes

- **F1** (fixed differently, see the frontmatter reason): `cmd/lym#testOnlyCodeAllowed` is added under `**Edit:**` on cards 1, 2, 3, 4, 6, 7, 8, 9, 10 and 11.
  Each creating card's Intent names its entries, keyed `<directory>.<Func>` or `<directory>.<Type>.<Method>`, each with a reason that names the card that adds the first caller and removes the entry.
  Each first-calling card's Intent names the entries it removes.
  The pairs, creating card first:
  - 1 → 3: `ParseMachineAddress` and `IsMachineAddress`.
  - 2 → 3: `machineOutboundEntry.message`.
  - 4 → 10: `IsDaemonRefusal`.
  - 6 → 10: `GoClient.TakeMachine`, `.ReportMachine`, `.SettleMachineDelivered` and `.SettleMachineReturned`.
  - 6 → 11: `GoClient.DeliverMachine`.
  - 7 → 10: `FormatBridgeMessage` and `BridgeChannelName`.
  - 7 → 11: `ParseBridgeMessage`.
  - 8 → 9: `ReadPeers`, `UpdatePeers`, `API.BotUsersNamed` and `API.UserAppID`.
  - 8 → 10: `ReadBridgeMap`, `UpdateBridgeMap`, `BridgeMap.ChannelOf` and `API.PostTimestamped`.
  - 8 → 11: `API.MemberChannels` and `API.PostThreadReply`.
  - 10 → 11: `programMailbox.DeliverMachine` and `bridgePoll.wake`.

  Card 10's `**Uses:**` gains `BridgeMap.ChannelOf`, which its step 3 needs.
- **F2:** card 4 creates `ErrDaemonRefusal` and `daemonRefusal.Is`.
  Its Intent states that `IsDaemonRefusal` is `errors.Is(err, ErrDaemonRefusal)`, and that a fake outside the package wraps `ErrDaemonRefusal` to stand for an answered refusal.
  Card 10 states that `fakeMailbox` scripts a refusal that way.
  Cards 10 and 11 list `ErrDaemonRefusal` under `**Uses:**`.
- **F3:**
  - Card 9 now states that `runPeerCommand` returns its failure as an error, which `handle` passes to `retryOrGiveUp`, and that it reads only `inbound` fields set at build time and writes none.
  - Card 11 step 2 states what happens when `runPeerCommand` fails: the holding cursor stays, the reader retries at each poll under its own failing key and first-failure time, and after `retryWindow` it answers and moves the cursor.
  - Card 11 step 2 also names which goroutine owns which state, and states that the reader posts through its `SlackClient`, never through `inbound.answer`.
  - `TestBridgeReceive` gains the `users.list`-refusal row, and card 11 lists `retryWindow` under `**Uses:**`.
- **F4:** card 5's Intent gains the `Bound:` lines after the `DeliverMachine` steps.
- **F5:** on card 10, `internal/slackengine/doc.go` and `pattern/PATTERN-daemon-wakeups.md` move to `**Edit:**`, and the empty `**Extends:**` is dropped.
  The Intent names the `doc.go` line to rewrite and the PATTERN bullet's final `and`.
- **F6:** card 9 compares the machine with this machine's names case-insensitively, and `TestInboundRead` gains the `Old_Box` / `approve old_box` row.
- **F7:** card 11's two refused-deliver rows are merged into one reply-to-refusal row, and the unreachable no-orch cause is dropped.
- **F8:** card 5 drops `Store.releaseGoneHolder` from `**Edit:**`.
  The Intent now says that machine-inbound mail already takes the undelivered branch, because its sender fails `isStrandName`, and that the `TestSweep_RemovedHolders` row pins it.
  The ImpactSummary no longer counts three widened guards.
- **F9:** card 6 lists `internal/creelcli#trustField` under `**Edit:**` and rewords its doc comment to cover another hub or another machine.

## Self-check

- fix-consistency, LOW: card 6's ImpactSummary still said every client method has no caller until card 10. Fixed: it now says four do, and `DeliverMachine` has none until card 11.
- fix-consistency, LOW: the `Old_Box` row needs fixed test names that hold `Old_Box`. Fixed: card 9 says the `fakeHub` and `newRelayRun` former names include `Old_Box`.
- fix-consistency and fix-affected-set, LOW (merged): card 11 step 2's own-machine check was still exact-case. Fixed: it now matches case-insensitively as card 9 does, and its row covers `old_box`.
- fix-proposal, LOW: the F3 give-up text did not cover an operator command written as a thread reply. Fixed: the give-up answers with `PostThreadReply` in the thread and moves that thread's cursor.
- fix-affected-set, LOW: card 9 claimed to be `ReadPeers`'s first caller, but no step of its Intent read the record. Fixed: `revoke` reads the record with `ReadPeers` before it writes.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
