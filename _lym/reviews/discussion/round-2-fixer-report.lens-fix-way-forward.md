```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking that every refusal, notice and next step the fix added can be carried out and makes progress.

**MEDIUM: the way forward for an unreadable hubs record leads back into the state F6 closed**
- **Changed text:** D3's added sentence: "Its way forward is moving that record aside, after which each hub records itself again at its next `lym creel inbox`." D5 step 2 repeats it ("or a failed `hubregistry.Live` read, refuses the same way (D3)").
- **Issue:** Once the record is moved aside, `hubregistry.Live` reads without error but lists only the hubs whose daemons have recorded themselves again. That gives two problems:
  - D1 treats events for a channel in a hub that has not recorded yet as "no live hub maps" while "every live map reads", so they are acknowledged and dropped.
  - D3's user-token test ("held by no other orch's entry in any live hub's map") passes against a partial list, which is the vacuous pass F6 set out to stop.
- **Evidence:** A hub records itself only from its creel daemon, inside `daemon.recordHub` (`internal/creelengine/hubs.go:72-92`), and only while running and at the `hubRecordInterval`. Nothing waits for every hub to record before the holder acts on the list again.
- **Effect:** The way forward can be carried out, but it opens a window in which the guard is bypassed and events are lost. The dropped messages come back only at the next start or reconnect read.

**LOW: a relay that the login ends has no stated way back while presence is terminal**
- **Changed text:** D3's added sentence: "A login that changes a stored token ends every live relay of the machine's live hubs, so a relay starts again only on the new tokens, through the routes that start one today."
- **Issue:** Dobby's restart is `EnsureRelayWhileAway`, which runs only while away. For a hub whose presence is terminal, nothing restarts the relay until an orch bring-up or `lym slack start`, and no notice tells the operator so.
- **Missing way forward:** The login output never names `lym slack start` for the hubs whose relays it ended. The D3 criterion added for this ("ends every live relay") checks only that the relays end, not how they come back.

**LOW: events left unacknowledged while the live-hub list fails are not caught up when it reads again**
- **Changed text:** D1's added clause "while the `hubregistry.Live` read fails, every event is" left unacknowledged, and its new acceptance-criterion line.
- **Issue:** D1's owed-resync mechanism covers a failed hint write, not a failed list read. Once Slack stops retrying an unacknowledged event, nothing writes a resync hint when the list reads again. The messages wait for the next reconnect, which comes every few hours by the Constraints' `disconnect` fact, or for a relay start.
- **Gap:** The fail-closed state has no step that leads back to delivery once it clears, against the Goal of about a second. Owing a resync hint to every live hub when the list reads again would close it.

Further tool calls:
- grep of `internal/creelcli` for the hub `Record` callers: needed to check when a moved-aside hubs record gets rewritten; my context lacked it.
- grep of `internal` for `.Record()` callers: to find where the daemon records its hub; my context lacked it.
- `sed -n 70,95p internal/creelengine/hubs.go`: to read `recordHub`'s trigger and interval; my context lacked it.
