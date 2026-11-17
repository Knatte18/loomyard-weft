# Card 2 — machine-mail-journal

**Create:**
- `plan:internal/creelengine#eventMachineOutbound` -> `const eventMachineOutbound = "machine-outbound"`
- `plan:internal/creelengine#eventMachineHeld` -> `const eventMachineHeld = "machine-held"`
- `plan:internal/creelengine#eventMachineDelivered` -> `const eventMachineDelivered = "machine-delivered"`
- `plan:internal/creelengine#eventMachineReturned` -> `const eventMachineReturned = "machine-returned"`
- `plan:internal/creelengine#eventMachineInbound` -> `const eventMachineInbound = "machine-inbound"`
- `plan:internal/creelengine#machineOutboundEntry` -> `type machineOutboundEntry struct`
- `plan:internal/creelengine#machineOutboundEntry.message` -> `func (o *machineOutboundEntry) message() Message`
- `plan:internal/creelengine#pool.applyMachineOutbound` -> `func (p *pool) applyMachineOutbound(rec record, day string)`
- `plan:internal/creelengine#pool.applyMachineInbound` -> `func (p *pool) applyMachineInbound(rec record, day string)`

**Edit:**
- `internal/creelengine#record`
- `internal/creelengine#Message`
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool`
- `internal/creelengine#newPool`
- `internal/creelengine#pool.apply`
- `internal/creelengine#pool.applyCarried`
- `internal/creelengine#pool.carryRecords`
- `internal/creelengine#pool.moveOrigins`
- `internal/creelengine#pool.prune`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine#TestJournal_Retention`
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/creelengine/doc.go`

**Uses:**
- `internal/creelengine#outboundState`
- `internal/creelengine#outboundPending`
- `internal/creelengine#outboundDelivered`
- `internal/creelengine#outboundReturned`
- `internal/creelengine#eventCarried`
- `internal/creelengine#firstLine`

**Intent:**
The journal form of D3 and D8: five new events, a map of their own in the pool, and the retention carry.
Nothing writes the events yet; cards 3 to 5 do, so this card fixes the form they write.

`record` gains `ReceivingMachine string` (`receiving_machine`), the machine a machine-outbound message goes to, `SendingMachine string` (`sending_machine`), the machine a machine-inbound message came from, and `SlackTS string` (`slack_ts`), the Slack timestamp of a delivered post, each `omitempty`.
The hold reason of `machine-held` and the reason of `machine-returned` use the existing `Reason` field.
`Message` gains `SendingMachine string`, empty for every message that is not machine-inbound; `LogEntry` gains `ReceivingMachine`, `SendingMachine` and `SlackTS`, filled by `logEntryFor` from the record.

`machineOutboundEntry` holds `ID`, `From` (the local orch), `To` (the full `<machine>::<name>` address), `ReceivingMachine`, `Kind`, `Body`, `ReplyTo` (the sending id on the other machine a reply answers), `Answers` (the local message it answers), `ExpectReply`, `Posted`, `State outboundState`, `HoldReason`, `Reason`, `SlackTS`, `Replied` and `origin`.
`pool` gains `machineOutbound map[int64]*machineOutboundEntry`, made by `newPool`; it is apart from `outbound`, so `pendingOutbound`, `outboundHubs` and the per-hub worker never see a machine-outbound message.
`message()` builds the `Message` a reply wait is answered with, as `outboundEntry.message` does.
No main reaches it until card 3's `settledOutcomeLocked` answers a wait with it, so `cmd/lym#testOnlyCodeAllowed` gains `internal/creelengine.machineOutboundEntry.message` with the reason `no caller until card 3 answers a reply wait with it; that card removes this entry`.

`apply` folds:
- `machine-outbound` through `applyMachineOutbound`: a pending entry, idempotent by id, and `Replied` set on the local message it `Answers` when that message expects a reply.
- `machine-held`: the entry's `HoldReason` becomes the record's reason.
- `machine-delivered`: the entry becomes delivered with the record's `SlackTS`, from pending or from returned.
- `machine-returned`: a pending entry becomes returned with the record's reason.
- `machine-inbound` through `applyMachineInbound`: unread mail in `messages`, as `applyInbound` folds inbound mail, with `SendingMachine`, `SendingID`, `Trusted`, `RemotePosted` and `DeliveredTo` from the record; a reply marks the machine-outbound entry it answers `Replied`.
Each joins the log.

`carryRecords` carries each pending machine-outbound entry older than the cutoff as `carried` with `of` `machine-outbound`, its receiving machine and its hold reason in `Reason`, and an unread machine-inbound message with `of` `machine-inbound`.
`applyCarried` folds both back, an entry already held left as the original lines made it; `moveOrigins` and `prune` treat `machineOutbound` as they treat `outbound`.
A binary that does not know these events or `of` values ignores them through the default branches of `apply` and `applyCarried`, so it reads each such message as an id alone through `maxID`; the row `an unknown key and an unknown event name are ignored` of `TestJournal_Fold` already pins that path for any unknown event or `of`.

`TestJournal_Fold` (tier 0) gains rows:
- a `machine-outbound` line folds into `machineOutbound` with its receiving machine and no entry in `outbound` or `messages`;
- `machine-held` lines set the hold reason, the later one standing;
- `machine-delivered` after `machine-returned` leaves the entry delivered with its Slack timestamp;
- a `machine-returned` line after `machine-delivered` changes nothing;
- a `machine-inbound` line folds as unread mail with its sending machine, and a machine-inbound reply marks the machine-outbound entry it answers replied.
`TestJournal_Retention`'s `mail across hubs` subtest gains the machine cases: a pending machine-outbound entry is carried with its receiving machine and last hold reason, a delivered or returned one is not, an unread machine-inbound message is carried with `of` `machine-inbound`, and a carried line beside its original folds to one entry.

`internal/creelengine/doc.go` gains a `# Mail across machines` section after `# Mail across hubs`, opening with the five events, the map of their own and the carry, in the words above.

**ImpactSummary:** Additive fields and fold branches in `internal/creelengine` alone; no writer exists yet, so no behaviour of a running daemon changes.

**Commit:** 2: creel folds and carries machine mail events
