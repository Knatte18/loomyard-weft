```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each fix covering a set (callers, sites or rows under one key) covers every member of that set.

Findings left unchecked: none. I reran the searches for F1, F2, F3, F5, F6 and F9. F4, F7 and F8 each concern a single site, not a set.

**LOW: card 11's own-machine check in a bridge channel is still case-sensitive (F6 fixed in one place only)**
- Changed text: card 9's Intent now says an `approve`/`revoke` naming this machine's current or a former name is refused "case-insensitively, as D6 matches a machine name".
- The sibling check in card 11 `handle` step 2 was not changed: "one naming this machine is left to the peer's relay".
- That check sees the same `parsePeerCommand` output, which has the machine name lower-cased.
- With former name `Old_Box`, `approve old_box` typed in a bridge channel never matches as "this machine". It falls outside both of step 2's branches, and card 11's row "an operator `approve` or `revoke` naming this relay's own machine … pins nothing and answers nothing" does not pin the former-name case.
- Evidence: `_lym/plan/11-bridge-receiving.md` step 2, and `_lym/plan/09-approve-and-revoke.md:38`, which lower-cases the machine.

**LOW: card 9's claim to be the first caller of `ReadPeers` rests on no step in its Intent**
- Changed text: card 9 now says "This card is the first caller of `ReadPeers`, `UpdatePeers`, `API.BotUsersNamed` and `API.UserAppID`, so it removes their `cmd/lym#testOnlyCodeAllowed` entries of card 8."
- No step in card 9's Intent reads the peers record: approve pins and revoke removes through `UpdatePeers`. Only the **Uses** list names `ReadPeers`.
- If card 9's code reaches the record only through `UpdatePeers`, removing the `ReadPeers` entry at card 9 leaves `ReadPeers` unreachable until card 10's `sendOne` step 2. The guard fails either way: an unreachable function with its entry removed fails just as a stale entry would.
- Evidence: `_lym/plan/09-approve-and-revoke.md` Intent (approve and revoke bullets); `_lym/plan/10-bridge-sending.md` step 2 "P's machine has no pin (`ReadPeers`)".

The rest of each set is covered:
- **F1:** every planned production function whose first caller comes in a later card has an allowlist entry and a removal, including the members the review missed. Those are `machineOutboundEntry.message` (card 2 → 3), `programMailbox.DeliverMachine` and `bridgePoll.wake` (card 10 → 11), and `GoClient.DeliverMachine` moved to card 11.
- **F9:** the other "another hub" texts in `cli.go` (lines 402 and 610) are the `postCmd` and `forwardCmd` help, which card 3 already edits.
- **F5:** in `doc.go`, only line 79 lists the loops.

Out of scope, noted only: card 11's resync-wanted mark is set by `wakeLoop.wake` and read by the poll goroutine, and no card names who owns it. The fix did not touch that text.

Tool calls:
- grep of every planned `func` Create line across the cards: my context had only the cards' Create lists one by one, and the F1 set needed all of them together.
- grep of the overview for case rules, `cli.go` for "another hub" and `doc.go`/PATTERN for loop lists: the sets for F6, F9 and F5 reach beyond the spans my context held.
- grep of the plan for "insensitiv" and D6: to find where D6's case rule is stated, which my context lacked.
