---
verdict: APPROVED
---

No findings.

The range is the one commit after the rejected head `302a1dba0` (`3456a3177`, card 17).

## Rejected

- Plain `read` selection: `Store.Read` takes `unread[0].From` as the oldest sender and keeps only that sender's messages.
  `unreadLocked` sorts by ID and excludes reserved and read messages, so the set is the oldest unread message plus every other unread message from its sender, and nothing else.
- `read --from` and `read --all`: with `from` set the filter is applied in `unreadLocked` and the plain-read narrowing is skipped; with `all` set no narrowing applies.
  The loop that appended read messages, and its re-sort, are removed, so no read message comes back.
- Marked read: `Reading.Ack` journals `eventRead` for exactly the reserved ids it handed out, and `Release` returns exactly those.
  The `Read_SetsAndInbox` sequence follows the card (interleaved `b1 a1 b2 a2 a3`, `Inbox` counts batten 2 and alpha 3 before the first read, and a repeated read returns nothing), which pins the repeated-read behavior.
- Tests: `lyx gate test` over `internal/creelengine` and `internal/creelcli` passes.
  `Read_SetsAndInbox` is rewritten rather than duplicated, and the single new row `ReadAllHandsOutEveryUnreadMessageFromEverySender` covers `All` crossing the protocol, which no other row covers; the dropped `read --all` assertion no longer holds.
  No test economy finding.
- Comments and docs: the `Reading`, `Store.Read`, `Request.All`, `readCmd` help and `doc.go` wording match the card.
  The grep for `still kept`, `still retained` and `unread ones` over `internal/creelengine` and `internal/creelcli` finds nothing left.
- Per-card mechanical check: the card has no `Rename` or `Delete` group, so no type-specific check applies.
