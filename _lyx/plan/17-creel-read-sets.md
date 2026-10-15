# Card 17 — creel-read-sets

**Edit:**
- `internal/creelengine#Store.Read`
- `internal/creelengine#Request`
- `internal/creelengine#Reading`
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
`Store.Read`'s doc comment, which says `all` adds the read messages still retained, names the three sets instead, and its second line (`The unread ones are reserved so a concurrent read by the same holder skips them`) says `They are reserved` instead, since every handed-out message is now unread.
`Reading`'s doc comment changes from "its unread ones are reserved until Ack or Release" to say the messages it hands out are the unread ones, reserved until Ack or Release.
`Request`'s `All` field comment says it makes read return every unread message instead of only the oldest sender's.
`readCmd`'s `Long` help and `--all` flag help name the three sets: `read` prints the oldest unread message and every other unread message from its sender, `--from` every unread message from one sender, `--all` every unread message; neither mentions read messages kept.
`internal/creelengine/doc.go`'s read paragraph states the same three sets in place of "the caller's unread messages … and --all adds the read messages still retained", and its second sentence (`The unread ones are reserved until the client acknowledges them …`) says `They are reserved` instead.
`TestStore_Mail`'s `Read_SetsAndInbox` row is rewritten to pin the three sets with this exact sequence, each read asserting exactly the bodies named.
A holder gets mail from two senders, posted interleaved: `b1` from batten, `a1` from alpha, `b2` from batten, `a2` from alpha and `a3` from alpha, so `Inbox` counts batten 2 and alpha 3 before the first read.
The oldest message's sender, batten, holds fewer unread messages than alpha, so a rule that picks the sender with the most unread messages fails the first read.
Plain read gives `b1`, `b2`, with `a1`, `a2`, `a3` left unread and still counted by `Inbox`; the next plain read gives `a1`, `a2`, `a3`; a further one gives nothing.
Post `a4` from alpha, `b3` from batten and `a5` from alpha.
A read with `from` set to batten, the sender that is not the oldest, gives only `b3`, leaving `a4` and `a5` unread.
Post `b4` from batten; a read with `from` set to batten and `all` set gives only `b4`: the old set would add the read `b1`, `b2` and `b3`, and without the narrowing it would add `a4` and `a5`.
Post `b5` from batten; a read with `all` set gives `a4`, `a5`, `b5`, and a further read with `all` set gives nothing, so no read message comes back.
`TestDaemon_Protocol`'s `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` row stays on `asStrand` true and only loses its last assertion, the `read --all` one that expects the acked message back.
A new row in the same table, `ReadAllHandsOutEveryUnreadMessageFromEverySender`, pins `All` crossing the protocol.
It runs with `asStrand` false, because on Linux a program claim from a pane that is not an orch strand's is refused, and with `asStrand` true every call from the test process resolves to alpha's pane.
It starts the daemon, has a client with `programClaim("ly:BATTEN")` send `VerbRead` once so the mailbox is known, then has a client with `externalClaim()` and a client with `programClaim("ly:CARTER")` each post one message to `"ly:BATTEN"`, and sends the BATTEN client a `Request` with `Verb` `VerbRead` and `All` true.
It asserts the bodies handed out are exactly the two posted ones, in post order: the row fails a plain read, which hands out one sender's body only, and so fails an `All` dropped before `Store.Read`.
The row holds no read message, so it does not tell the old read-plus-unread set apart; the `Read_SetsAndInbox` sequence is what pins that no read message comes back.
`TestSweep_RemovedHolders`' `holderOf` helper reads with `all` set and releases, and sees only unread messages, which is all it asserts on; it stays as it is.
The grep for the old wording (`still kept`, `still retained`) over `internal/creelengine` and `internal/creelcli`, and for `unread ones` over `internal/creelengine`, hits only the sites this card edits: `verbs.go`'s `Reading` and `Store.Read` comments, `protocol.go`'s `Request.All` comment, `cli.go`'s `readCmd`, `doc.go` and `server_test.go`'s `TestDaemon_Protocol` message.

**ImpactSummary:** Changes which messages one read hands out and reserves; the read verb's signature, protocol shape and CLI flags stay, and only the creel engine, its CLI help and two test rows change.
