# Card 5 — expect-reply-engine

**Create:**
- `internal/creelengine#PostOptions`
- `internal/creelengine#Store.PostWith`

**Edit:**
- `internal/creelengine#Request`
- `internal/creelengine#record`
- `internal/creelengine#Message`
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool.applyPosted`
- `internal/creelengine#pool.applyCarried`
- `internal/creelengine#pool.carryRecords`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#MessageLine`
- `internal/creelengine#LineFits`
- `internal/creelengine#Store.appendPostedLocked` -> `func (s *Store) appendPostedLocked(sender Sender, rec record) (int64, error)`
- `internal/creelengine#Store.post` -> `func (s *Store) post(sender Sender, receiver, body string, opts PostOptions) (int64, bool, *Waiter, error)`
- `internal/creelengine#Store.storePost` -> `func (s *Store) storePost(sender Sender, receiver, body string, wait, expect bool) (int64, *Waiter, error)`
- `internal/creelengine#Store.Post`
- `internal/creelengine#Store.PostWait`
- `internal/creelengine#Store.PostInterrupt`
- `internal/creelengine#Store.Reply`
- `internal/creelengine#Store.PostUnchecked`
- `internal/creelengine#Store.settledOutcomeLocked`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine#TestJournal_Retention`
- `internal/creelengine/doc.go`

**Uses:**
- `internal/loomcli#noticeFits`

**Intent:**
A post can carry an expected-reply mark (D9), journaled with the post and cleared by the holder's reply.

- `Request` gains `ExpectReply bool` (`json:"expect_reply,omitempty"`), and `record` the same field, written on a `posted` record and on a `carried` snapshot of a message.
- `PostOptions` holds `Wait`, `Interrupt` and `ExpectReply`; `Wait` implies the mark.
  `PostWith` is the one entry that takes them: `post` and `storePost` take the options and the mark through, and `Post`, `PostWait` and `PostInterrupt` stay as wrappers that call `post` with the options they stand for, so their callers and signatures are unchanged.
  `daemon.dispatch`'s `VerbPost` case calls `PostWith` with the request's `Wait`, `Interrupt` and `ExpectReply`.
- `appendPostedLocked` takes the sender and a `record` carrying `To`, `Kind`, `Body`, `ReplyTo` and `ExpectReply`, and fills the event, id, sender and envelope itself, so a later card adds the chase fields without another re-sign; `storePost`, `Reply` and `PostUnchecked` (in `export_test.go`) build that record.
- `Message` gains `ExpectReply bool` and `Replied bool`.
  `applyPosted` sets `ExpectReply` from the record; a `posted` reply record whose `ReplyTo` names a marked message sets that message's `Replied`, since `Reply` accepts a reply only from the message's current holder, so the expectation follows the holder through a forward.
  `applyCarried` reads the mark from a carried snapshot and `carryRecords` writes it, so retention keeps an unread marked message's mark.
- `LogEntry` gains `ExpectReply bool` and `ReplyTo int64`: `logEntryFor` sets `ExpectReply` on a `posted` or `carried` entry of a marked message, and `ReplyTo` on the entry of a reply, the id of the message it answers (D11).
- `MessageLine` renders a marked message as `[creel] #<id> from "<sender>", expects a reply (lym creel post --reply <id> "<text>"): <body>`, so the hook-delivered line and a typed notice show the mark beside the reply command; an unmarked message's line is unchanged.
  `LineFits` measures the marked form, the longer one, so a post's size check never depends on the mark.
  Its other caller, `internal/loomcli`'s `noticeFits`, then measures an unmarked `lym loom decision add` notice against the marked form too, so a notice within the mark's length of the 8 KiB limit takes the record-pointing form: a conservative bound no notice post can exceed, and intended.
- `Reply` to a message whose sender is not EXTERNAL also answers a wait registered on the original with `OutcomeReply` carrying the reply, besides delivering the reply as mail, so a strand's or program's `post --wait-reply` ends with the reply as the creel doc's Mail section already states ("A waiting post … ends with the reply or with the message returned undelivered"); with no wait registered nothing changes.
- `settledOutcomeLocked`'s reply-wait case also answers a wait on a message whose sender is not EXTERNAL with `OutcomeReply` carrying the oldest reply whose `ReplyTo` is the id, so a reconnecting `--wait-reply`, which `Client.resume` registers as `VerbWaitReply`, gets a reply that landed while it was away; that reply stays mail to the strand, so no `delivered` record is journaled for it.
  Card 7's unanswered case comes after it.

Tests:
`TestStore_Mail` gains rows: a `PostWith` with `ExpectReply` hands out a message whose `ExpectReply` is set and whose `posted` log entry carries it; a `PostWith` with `Wait` and no `ExpectReply` carries the mark too; an unmarked post carries none; a reply by the holder carries `ReplyTo` on its log entry; a strand's `PostWait` is answered `OutcomeReply` with the reply when the holder replies, and the reply still reaches the strand as mail; a strand's `PostWait` is cancelled, the holder replies, and a `WaitReply` registered afterwards is answered `OutcomeReply` with that reply.
Its row `Post_RefusesEmptyAndOversizedBody` takes its overhead from the marked `MessageLine` form, so its exact-fit body still posts, and gains a body that fits the plain line but not the marked one, which is refused.
`TestStore_ClaimAtBoundary` gains a row asserting the claimed line of a marked message equals the marked `MessageLine` form, and an unmarked one the plain form.
`TestDaemon_Protocol` gains a row posting with `expect_reply` over the socket and reading the mark back from the `log` verb's entry.
`TestJournal_Fold` gains a row folding a marked `posted` record and a reply to it, asserting `ExpectReply` and `Replied`, then a second marked `posted` record, a `forwarded` record and a reply by the new holder, asserting `Replied` on that message too, and `TestJournal_Retention` a row asserting a carried unread marked message keeps its mark.
`internal/creelengine/doc.go`'s Mail section gains a paragraph on the mark: set by `--expect-reply` or implied by `--wait-reply`, shown in the line, cleared by the holder's reply; its line form and its Tool-boundary sentence on `MessageLine` name the marked form beside the plain one.

**ImpactSummary:** Post gains an options entry and a journaled mark; Post, PostWait and PostInterrupt keep their signatures, and the one re-signed internal helper's three callers are all on this card; a reconnecting reply wait now gets a standing reply, and `loomcli`'s decision notices are measured against the marked line, a few dozen bytes tighter.
