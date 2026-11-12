```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: traced each decision's new state against the code that already runs. Findings below.

**Findings**

1. **MEDIUM — D3, outbound map: creel's own outbound worker would pick up machine-bound mail.**
   - Machine-outbound entries go into the same outbound map. Creel's `Scheduler` starts one worker per receiving hub for every pending entry: through `Store.outboundHubs` when it starts, and through `wakeOutbound` on each post. That worker calls `deliver` on that hub's daemon.
   - A machine-outbound entry has no receiving hub. The worker would retry `deliver` for the whole hour, then return the message with a reason such as `hub-unreadable`, even though the relay already posted it. Two settlers would then race on the same id.
   - The record only says the post "never reaches the machine record or the `deliver` and `confirm` verbs". It never says that the scheduler's start scan, `wakeOutbound` and `pendingOutbound` skip machine entries, or that the relay's take verb is the only reader of them.
   - Evidence: `internal/creelengine/outbound.go` `outboundHubs` collects `entry.ReceivingHub` for every entry whose state is `outboundPending`, with no other filter. The package doc says "a worker starts … for each hub with a pending message, when the Scheduler starts".

2. **MEDIUM — D8, holding the cursor: the existing failure path gives up after five minutes.**
   - The relay's existing path for a message it cannot handle is `inbound.retryOrGiveUp`. It holds the cursor only until the message's first failure is `retryWindow` old. It then answers in Slack and reports the message handled, so the cursor moves past it.
   - It also tracks a single `failingKey` for the whole relay.
   - D8 needs a held message (peer not approved, wrong app, or the receiving orch's button off) to stay held until approval or until the button is on. If the bridge read reuses this machinery, every held message is silently skipped after five minutes, and holds in two channels reset each other's key.
   - The record does not say that a bridge hold is outside `retryOrGiveUp` and its single key.
   - Evidence: `internal/slackengine/inbound.go`, the doc comment of `retryOrGiveUp` and the body around lines 698–714.

3. **MEDIUM — D8, holder routing: the holder never sees the message text.**
   - D8 has the holder route an event in an unmapped channel "by the event's envelope", meaning the `[lym-bridge] … to=` line. The holder's decoded `Envelope` keeps only `Type`, `EnvelopeID`, `Reason`, `Channel`, `TS`, `ThreadTS` and `Subtype`, with no text.
   - `route` reads only the hubs' channel maps and drops the event when none holds the channel ("dropped an event no hub's channel map holds"). The hint name it writes is the channel id alone, and the hub relay's `readsForHint` resolves it against the orch channel map.
   - So routing by envelope needs either the event's text decoded in the socket layer, or a history read in the holder before it routes. The record names neither, so the mechanism as stated has no input to work on.
   - Evidence: `internal/slackengine/socket.go` `Envelope`; `internal/slackengine/holder.go` `route` (lines 329–354) and `hintName`.

4. **MEDIUM — Constraints, older binaries reading the journal: "reads an `inbound` event as before" is a misread, not a safe fallback.**
   - The fold builds the message from `SendingHub`, `SendingRepository` and `Trusted`, and `Message.HubStamp` picks `StampTrusted` from `Trusted` alone.
   - A binary that does not know the machine fields would show a trusted machine message as `(other hub, trusted orch)`, which is the wrong origin.
   - Its `post --reply` would also take the cross-hub reply path to a `SendingHub` that is empty.
   - The record treats this as harmless instead of bounding it, for example by writing machine inbound under an `of` older binaries read as an id only, as carried inbound is.
   - Evidence: `internal/creelengine/fold.go` lines 93–113 and 531.

5. **LOW — D4, archiving: the peer's side of an archived bridge is unstated.**
   - Retirement through `keeper.retire`/`retireEntry` renames the channel to `ArchivedName` and archives it, which frees the computed name.
   - The peer's bridge map still holds the old channel id. Its next post would be refused five times (archived channel) and returned, instead of creating a new channel.
   - The record says nothing about the peer dropping an archived entry from its map.
   - Evidence: `internal/slackengine/keeper.go` lines 387–472.

6. **LOW — D10/D3, button switched off after a post was accepted: what happens to the pending message is unstated.**
   - The existing outbound read drops orch mail whose button is off.
   - For bridge mail that is already pending, the record does not say whether a relay that is still running (because another orch's button is on) posts it, holds it or returns it.
   - If this orch was the hub's last button, the relay ends, and the message waits out its hour and comes back as an unexplained return.

7. **LOW — D3, settle verb: the Slack timestamp does not fit the existing settle.**
   - D3 settles `remote-delivered` with the Slack timestamp, a string.
   - The existing `Store.settleDelivered(id, remoteID int64)` and the `remote-delivered` event carry the receiving hub's integer id, so the event's meaning changes for machine entries.
   - Evidence: `internal/creelengine/outbound.go` line 125.

**Paths traced**

- D1 parser: `agentname.Parse` and `ParseProgram` reject `::`, so no existing path misreads the new address. Survives.
- D2 refusals on `inject`, `interrupt`, `hold`, `release` and `forward`: these mirror the existing other-hub refusals. Survives.
- D3 creel outbound map, start scan and `wakeOutbound` worker: breaks (finding 1).
- D3 settle and `remote-delivered`: survives with a field-type change (finding 7).
- D3 post-time button check against the relay's existing drop of off-button mail: gap (finding 6).
- D4 keeper create, adopt and retire: archiving survives on this side; the peer's map is stale (finding 5).
- D6 operator commands in `inbound.handle`: `away`, `back` and `status` already skip forwarding and the presence-setting step, so `approve` and `revoke` handled the same way survive.
- D7 slackengine outbox dedupe: survives.
- D8 holder `route` and `hintName`: breaks (finding 3).
- D8 `readsForHint` against the orch channel map: needs the bridge map consulted; the record's "adds the channel to its map" covers it.
- D8 `retryOrGiveUp` and its single `failingKey`: breaks (finding 2).
- D8 inbound journal: dedupe survives in the new binary; an older binary misreads it (finding 4).
- D9 `Message.HubStamp`: needs the machine field consulted; the record covers the new binary only (finding 4).
- D10 relay lifetime under the buttons: survives.
- D11 `slack login` replacing the tokens with another app's: peers would hold mail under D8's "mismatched app" notice. Survives.

**Tool calls**

- Read `holder.go`'s function list and routing: my context lacked how the holder routes an event to a hub.
- Read `creelengine/outbound.go`'s function list: my context lacked how outbound workers are keyed and started.
- Read `inbound.go` around `retryOrGiveUp`, `keeper.go`'s archive and retire lines, and `fold.go`'s stamp fields: my context lacked the give-up window, the retire trigger and the inputs to the stamp.
- Read `socket.go`'s `Envelope`, `holder.go`'s `hintName` and `outbound.go`'s `outboundHubs`: my context lacked whether the event envelope carries the message text, and the outbound hub scan.
