# Card 3 — cross-hub-records

**Create:**
- `internal/creelengine#eventOutbound`
- `internal/creelengine#eventRemoteDelivered`
- `internal/creelengine#eventOutboundReturned`
- `internal/creelengine#eventInbound`
- `internal/creelengine#eventDropped`
- `internal/creelengine#outboundEntry`
- `internal/creelengine#outboundEntry.message`
- `internal/creelengine#pool.applyOutbound`
- `internal/creelengine#pool.applyInbound`
- `internal/creelengine#StampTrusted`
- `internal/creelengine#StampUntrusted`
- `internal/creelengine#Message.HubStamp`

**Edit:**
- `internal/creelengine#record`
- `internal/creelengine#Message`
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool`
- `internal/creelengine#newPool`
- `internal/creelengine#pool.apply`
- `internal/creelengine#pool.applyCarried`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#pool.carryRecords`
- `internal/creelengine#pool.moveOrigins`
- `internal/creelengine#pool.prune`
- `internal/creelengine#MessageLine`
- `internal/creelengine#composeReminder`
- `internal/creelcli#messageFields`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#writeMessagesText`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine#TestJournal_Retention`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelengine_test#TestStore_ClaimAtBoundary`
- `internal/creelcli#TestWriteMessagesText`
- `internal/creelcli#TestCreelCLI`
- `internal/creelengine/doc.go`

**Extends:**
- `internal/creelcli/doc.go`

**Intent:** Teach the journal and every rendering what a message from another hub is, before any code writes one, so the later cards only journal records this card already folds and shows (D2's, D3's and D6's records and stamp).

`record` gains `SendingHub`, `SendingShortname`, `SendingID`, `ReceivingHub`, `RemoteID`, `Trusted`, `RemotePosted`, `DeliveredTo` and `Answers`, each `omitempty` or `omitzero`.
`Message` gains `SendingHub`, `SendingID`, `Trusted`, `RemotePosted` and `DeliveredTo`; `SendingHub` is empty on every local message.
`DeliveredTo` is the receiver the delivery named, which never changes, since `Holder` moves on a forward or a sweep's return and D3's duplicate check compares the receiver of the delivery.
`LogEntry` gains `SendingHub`, `SendingShortname`, `SendingID`, `ReceivingHub`, `RemoteID` and `Answers`.

`pool.outbound` maps an id to an `outboundEntry`: id, sender, receiver, receiving hub path, kind, body, the id it answers on the other hub (`ReplyTo`), the local id it answers (`Answers`), expects-reply mark, post time, state (pending, delivered, returned), remote id, return reason, whether a reply came, and the journal day.
An outbound message lives only there and never in `pool.messages`, so no mail flag, delivery worker, reminder, sweep, chase or settle acts on it, since each reads `messages` alone.
`outboundEntry.message` builds the `Message` a wait is answered with: `Holder` the receiver and `Status` `StatusUndelivered` once returned.

`pool.apply`:
- `eventOutbound` runs `applyOutbound`; an entry that answers a local message (`Answers` not 0) also sets that message's `Replied` when it expects a reply, as `applyPosted` does for a local reply, so its chase stops.
- `eventRemoteDelivered` marks the entry delivered with `RemoteID`, and `eventOutboundReturned` marks it returned with `Reason`.
- `eventInbound` runs `applyInbound`, which builds the `Message` as `applyPosted` does for mail and sets the new fields; a reply's inbound record also sets `Replied` on `outbound[ReplyTo]`.
- `eventDropped` folds no entry and only joins the log.
- An older binary reaches `apply`'s `default` for all five and keeps only the id, which the fold's first line takes.

`logEntryFor` shows an outbound event with its sender, receiver, subject and `ReceivingHub`, an inbound event with `SendingHub` and `SendingID`, and a dropped event with the claimed strand as `From`, the receiver as `To`, `SendingHub`, `SendingShortname` and `Reason`, so `lym creel log --from <claimed strand>` finds it.
An outbound, an inbound and an `Of: eventInbound` carried entry carry `Kind`, `ExpectReply` and `ReplyTo` as a posted entry does, and an outbound entry its `Answers`.
`eventRemoteDelivered` and `eventOutboundReturned` entries take their parties, subject and `ReceivingHub` from `outbound`, so `lym creel log --id <id>` shows what D2's notice sends the sender to.

Retention: `carryRecords` carries a pending outbound entry with `Of: eventOutbound` and `OrigAt` its post time, to the nanosecond, and never a delivered or returned one.
An inbound message is carried under the existing `carriedMessage` rule with `Of: eventInbound`, never `Of: eventPosted`, since an older binary's `applyCarried` would fold `posted` into unstamped mail, and D2's bound has it fold nothing but the id.
`applyCarried` rebuilds an outbound entry with its sender, receiver, `ReceivingHub`, kind, body, `ReplyTo`, `Answers`, mark and post time.
It rebuilds an inbound message with every field the `Of: eventPosted` branch keeps, `ReadAt`, the chase clock and `Chases` included, and `SendingHub`, `SendingID`, `Trusted`, `RemotePosted` and `DeliveredTo`.
An unknown `Of` still takes its default branch, and a carried line beside its original folds to one entry.
`moveOrigins` and `prune` treat `outbound` as they treat `injects`.

`MessageLine` and `composeReminder` carry the stamp (D6).
`HubStamp` answers `StampTrusted`, `StampUntrusted`, or the empty string for a message with no `SendingHub`.
The line reads `[creel] #<id> from "<sender>" <stamp>: <body>`, and the marked form puts the stamp before `, expects a reply`.
A reminder entry reads `#<id> from "<sender>" <stamp>`.
A local message's line is unchanged byte for byte.

`writeMessagesText` puts the stamp after the quoted sender in the header.
`messageFields` keeps `from` the bare name and adds `trust` (`trusted-orch` or `untrusted`), `sending_hub` and `sending_id` for such a message, and every JSON rendering goes through it, `read --envelope` and the `reply` field of `post --wait-reply` included.
`logEntryFields` adds `sending_hub`, `sending_shortname`, `sending_id`, `receiving_hub`, `remote_id` and `answers` when set.

Tests, each row writing its journal lines by hand, since no code writes these events yet:
`TestJournal_Fold` rows: an outbound record folds into no message; delivered and returned settle it, and their log entries carry the parties, subject and `ReceivingHub`; a reply's outbound marks the local message replied; an inbound record folds as unread mail with its hub fields and `DeliveredTo`, and its log entry carries `Kind`, `ExpectReply` and `ReplyTo`; a forward of it moves `Holder` and leaves `DeliveredTo`; an inbound reply marks the outbound entry replied; a dropped record joins the log with the claimed strand as `From`.
`TestJournal_Retention` rows: a pending outbound is carried with its post time to the nanosecond and folds back with its `ReceivingHub`, `ReplyTo`, `Answers` and mark; a delivered and a returned one are not; an unread inbound message is carried `of: inbound` and folds back with its fields; a read, chaseable inbound message keeps its `ReadAt`, chase clock and `Chases`; an original line beside its carried line folds to one entry, for an outbound and an inbound message.
`TestScheduler_Reminders` rows: a pane-mail strand gets an inbound message from a strand reminded, not typed, with the stamp after the sender.
`TestStore_ClaimAtBoundary` rows: the trusted, the untrusted and the expects-reply line in full, and a local line unchanged.
`TestWriteMessagesText` rows: the stamp in the header.
`TestCreelCLI` rows, tier1, over a journal written before `startDaemon`: `read --envelope` keeps `from` bare beside `trust` and `sending_hub`, and `log` shows the sending hub's path and id on the message's entry.

**ImpactSummary:** Additive to the journal and to `Message`, whose field-by-field consumers in `orchcli` and `reportengine` read existing fields only; the five new events reach no flag, worker or sweep, and local lines stay byte-identical, so existing exact-line tests hold.
