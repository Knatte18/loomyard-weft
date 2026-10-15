# Card 17 — creel-read-sets

**Edit:**
- `internal/creelengine#Store.Read`
- `internal/creelengine#Request`
- `internal/creelcli#readCmd`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestDaemon_Protocol`

**Uses:**
- `internal/creelengine_test#env.readBodies`
- `internal/creelengine_test#wantBodies`

**Intent:** Make `read`, `read --from` and `read --all` return the sets the board's Reading section names, as D8 and its acceptance criterion require, replacing the sets the retired card 7 described.
`Store.Read(sender, from, all)` keeps its signature and hands out, oldest first, among the holder's unread messages no other read has reserved:
- plain `read` (no `from`, no `all`): the oldest such message together with every other such message from that message's sender, so one sender's thread is read at a time and the other senders' mail stays unread;
- `read --from <sender>`: every such message from that sender;
- `read --all`: every such message, from every sender, and never a read message; with `--from` as well, it narrows to that sender as `--from` alone does.
It reserves exactly the messages it hands out, so `Ack` marks exactly those read and `Release` returns exactly those; the loop that appended read messages for `all`, and its re-sort, go.
`read --wait` still waits until the caller holds unread mail and then reads as plain `read` does, through the same `Store.Read`.
`Request`'s `All` field comment says it makes read return every unread message instead of only the oldest sender's.
`readCmd`'s `Long` help and `--all` flag help name the three sets: `read` prints the oldest unread message and every other unread message from its sender, `--from` every unread message from one sender, `--all` every unread message; neither mentions read messages kept.
`internal/creelengine/doc.go`'s read paragraph states the same three sets in place of "the caller's unread messages … and --all adds the read messages still retained".
`TestStore_Mail`'s `Read_SetsAndInbox` row is rewritten to pin the three sets: a holder with unread mail from two senders, posted interleaved (`a1` from alpha, `b1` from batten, `a2` from alpha), gets `a1`, `a2` from plain read with `b1` left unread and still counted by `Inbox`, then `b1` from the next plain read; a fresh set read with `--from` returns only that sender's messages; `--all` returns every unread message from both senders and, once everything is read, returns nothing, so no read message comes back.
`TestDaemon_Protocol`'s `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` row's last assertion changes from `read --all` returning the acked message to returning none.
`TestSweep_RemovedHolders`' `holderOf` helper reads with `all` set and releases, and sees only unread messages, which is all it asserts on; it stays as it is.
The grep for the old wording (`still kept`, `still retained`) finds no other test asserting it.

**ImpactSummary:** Changes which messages one read hands out and reserves; the read verb's signature, protocol shape and CLI flags stay, and only the creel engine, its CLI help and two test rows change.
