# Card 6 — post-to-another-hub

**Create:**
- `internal/creelengine#Store.otherHub`
- `internal/creelengine#Store.refuseOtherHub`
- `internal/creelengine#Store.postOtherHub`
- `internal/creelengine#widestStampedLine`
- `internal/creelengine#validateStampedLine`

**Edit:**
- `internal/creelengine#Store.post`
- `internal/creelengine#Store.inject`
- `internal/creelengine#Store.Interrupt`
- `internal/creelengine#Store.Hold`
- `internal/creelengine#Store.Release`
- `internal/creelengine#Store.WaitReply`
- `internal/creelengine#Store.settledOutcomeLocked`
- `internal/creelcli#postCmd`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestStore_MailFlags`
- `internal/creelengine_test#TestSweep_RemovedHolders`
- `internal/creelengine/doc.go`

**Extends:**
- `internal/creelcli/doc.go`

**Uses:**
- `internal/testkit/creelfake#Hubs`
- `internal/creelengine#HubDirectory`
- `internal/creelengine_test#env.writeJournal`
- `internal/creelengine_test#env.reopen`

**Intent:** Accept a post to a strand of another hub and queue it in the sending daemon's journal (D2's post half), and refuse every other verb that names another hub (D8).
This opens a new routing edge: a strand may now put a body on a queue that leaves its hub.
It is bounded to a strand sender and a strand receiver, a receiver hub D1 resolves, and the stamped line cap, and nothing is delivered until card 8.

`otherHub` parses a name with `agentname.Parse`, then `agentname.ParseProgram`, and answers its shortname when that is not `s.geom.Shortname`; a name neither parses is no name of another hub, and takes today's unknown-receiver refusal.

`refuseOtherHub` answers `creel: <name> is a name of another hub, and <verb> stays inside one hub; way forward: lym creel post <name> "<text>"`.
`Store.inject`, `Store.Interrupt`, `Store.Hold` and `Store.Release` call it first, and `Store.post` calls it when `opts.Interrupt` is set, before any key can be sent.
It needs no seam, so it holds with a nil `HubDirectory` too.
`TestStore_InjectAndHold` rows reach each one with a receiver `zz:orch`: `inject`, `interrupt`, `hold`, `release` and `post --interrupt` each name `lym creel post` and leave nothing journaled.

`Store.post`, with a seam set and `otherHub` true, goes to `postOtherHub`, which refuses in this order, each row of `TestStore_Mail` reaching one with the fake `Hubs`:
1. A program or EXTERNAL sender: `creel: only a strand posts to another hub, and <sender> is none; way forward: post from a strand pane, or run lym creel post from the receiving hub's directory`.
2. A program or EXTERNAL receiver of another hub: the refusal names a strand of that hub as the way forward, `<shortname>:orch` by example.
3. `hubs.Lookup` failing: its text unchanged, D1's for an unknown shortname and for a shared one, and the record's own for a record that cannot be read. The call runs before the store lock.
4. A line over the cap: `validateStampedLine` measures `widestStampedLine`, which is `MessageLine` of a message with `math.MaxInt64` as id, `ExpectReply` true, a `SendingHub` set and `Trusted` false, the longer stamp, with the refusal `validateLine` words. A row posts a body whose local line fits and whose stamped line does not.

An accepted post journals `eventOutbound` under `newID` with sender, receiver, `ReceivingHub`, body, kind mail and the mark (`opts.ExpectReply || opts.Wait`), registers the wait when `opts.Wait` is set, and answers the id, so `dispatch` needs no change: `post --wait-reply` already reads the waiter.

`Store.WaitReply` accepts an outbound message whose sender is the caller.
`settledOutcomeLocked`'s `waitReply` case also reads `outbound`: a returned entry answers `OutcomeReturned` with `outboundEntry.message()`, and a stored reply answers as a reply to a strand's message does, the oldest reply whose `ReplyTo` is the id, since card 9's stored reply sits in `messages`.

`postCmd`'s `Long` says a receiver may be a strand of another hub by its full name, that such a post may wait up to an hour and returns with a `CREEL` notice, and `internal/creelcli/doc.go` says the same.

`TestStore_Mail` rows also pin the accepted post: an id at once, `outbound` in the log with the receiving hub's path, nothing in the sender's `Inbox`, and a `PostWait` whose waiter stays open.
Further rows, over journal lines written before a `reopen`, reach the wait branches: `WaitReply` on a pending outbound id by its sender stays open, as the `--wait-reply` reconnect resumes it; by another strand it is refused; on a returned entry it answers `OutcomeReturned` at registration; on an entry with a stored reply it answers that reply at registration.
`TestStore_MailFlags` row: a pending outbound message raises no flag.
`TestSweep_RemovedHolders` row: under a started scheduler with `zz`'s geometry failing, a pending outbound message to `zz:orch` makes no sweep subject; the row moves the clock past two sweep ceilings with one `Advance`, never `sweepStep`, since no sweep timer is armed and card 8's retry waits share the sweep's range, and asserts that nothing was typed and the log holds neither `undelivered` nor `outbound-returned`.

**ImpactSummary:** All new behaviour sits behind a name of another hub; `Store.post`, `inject`, `Interrupt`, `Hold` and `Release` keep their signatures and local paths, and the four refusals replace only the unknown-strand refusal those verbs gave such a name, which no test asserted.
