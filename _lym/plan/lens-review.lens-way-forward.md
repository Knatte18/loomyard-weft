```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: every refusal, stop and recovery step the cards add, and whether its next step can succeed. Refusals not listed under Findings progress.

**Findings**

1. MEDIUM, card 1 and card 4 (D1 none refusal, corrupt-record refusal): the way forward "run any lym creel verb, such as lym creel inbox, in that hub once" fails when that hub's daemon is already live.
   - Evidence: card 4 calls `Record()` only inside `Run`, after the lock. A verb against a live daemon starts nothing (`internal/creelengine/client.go:75-79`).
   - States it strands:
     - Card 1's corrupt-record way forward removes the file, and every live daemon then stays unrecorded. "Each hub's next daemon start" has no verb that causes it.
     - Card 4's failed record logs one Warn and serves on, with no retry.
   - Proposed change: make D1's sentence true. A served local verb re-records when the last `Record` failed or the hub's own entry is missing, throttled, with a `TestDaemon_Lifecycle` row and a `TestRecord` row.

2. MEDIUM, card 7 step 2 (Lookup failure at the receiver): any `hubs.Lookup` error answers `refused`, so a corrupt or unreadable record returns the message at once.
   - Evidence: card 7 step 2 reads "a refusal answers `refused` with D1's text". Card 1 has `Lookup` refuse a corrupt record through the same single `error`, so the engine cannot tell it from D3's none or several. The `HubDirectory.Lookup` signature on card 4 is `(string, error)`.
   - D3 step 2 names only none and several as refusals, and D3 says a failed step is transient. A repairable registry fault becomes a final return, even though the sender could retry for the hour.
   - Proposed change: add a typed none/several error from `hubregistry` that the seam passes through. Any other error is returned, which is the transient `OK: false`, with a row in `TestDaemon_CrossHubDeliver`.

3. MEDIUM, cards 7 and 8 (id-clash way forward against the 200-byte cut): the clash refusal's way forward can be cut off before it reaches the sender.
   - Evidence: card 7 step 3's reason names the clash, the range and "a post succeeds once the sending hub's ids pass that range or the held messages pass the seven-day retention". Card 8's `quoteAnswer` cuts to 200 bytes.
   - With a hub path or strand names in the text, the way forward is the part lost, and D3's bound says each return carries that reason. D1's several refusal, which names every hub path, is cut the same way.
   - Proposed change: fix the clash text so the way forward comes first or the whole reason is under 200 bytes. Add a row asserting that the quoted reason still names the retention and the range.

4. MEDIUM, card 8 (failed settle after an answer): the card gives no path for a journal write that fails in `settleDelivered` or `settleReturned`.
   - Evidence: card 8 reads "`accepted`: `settleDelivered` …, and the worker goes straight to the next entry" and "`refused`: `settleReturned` at once". The oldest pending entry is then the same entry, so the worker re-delivers with no wait. That loop breaks the one-second floor (`pattern/PATTERN-daemon-wakeups.md`, The rule) and hammers the receiver while the disk stays full.
   - Card 7's `dropLocked` likewise has no stated answer when its journal write fails, although the operator decision requires each drop to be journaled.
   - Proposed change: a failed settle takes the transient backoff and logs once per episode. A failed drop journal returns an error, which is transient. Each gets a row using the fault seam of finding 5.

5. MEDIUM, card 7 row "A receiver that confirmed and died before storing": the recovery row cannot be written as stated.
   - Evidence: `FailJournalWrites` is a `Store` method (`internal/creelengine/export_test.go:21`). `Run` builds its store internally (`internal/creelengine/server.go:78`), and the rig holds no store.
   - A write fault also does not stop a daemon: it answers an error. No card lists an `export_test.go` hook.
   - Proposed change: add a `With…` export beside `WithBeforeOutcomeWait` on card 7, or drive the row through `Store.Deliver` directly with a fake confirm. Reword the row as a failed store followed by a retry.

6. LOW, card 4 against card 7 (nil seam): the "fail-loud wiring guard no entry point reaches in production" is reachable, and it names no next step.
   - Evidence: card 4 has `serveDaemon` leave `Hubs` nil when `registryDir` fails. A daemon recorded earlier, or any hand-written `deliver`, then reaches it.
   - The same daemon answers a post to `lyt:orch` with "unknown receiver" and nothing about the cause.
   - Proposed change: drop the claim and name the state directory cause in both answers, or refuse the daemon's cross-hub verbs with that text.

7. LOW, card 1 (D1 several refusal): "needs distinct shortnames" names no step that can succeed in place.
   - Evidence: `lym fabric shortname` refuses to change a recorded shortname (`internal/fabriccli/shortname.go:39-43`).
   - D1 fixes the wording, so the card should add the real route, or say that none exists without re-cloning.

8. LOW, card 8 (start cap): a hub past `maxHubStarts` returns every later post with `connect-failed`, and the notice's `lym creel log --id` shows no cause.
   - Evidence: `Client.gaveUp` quotes the daemon log (`client.go:119-122`), and the code drops that text. The count survives worker retirement and resets only on an answer or a sending daemon restart.
   - The sender cannot tell a capped hub from a down one.
   - Proposed change: add a code such as `start-capped`, plus one Warn naming the hub's daemon log path.

9. LOW, card 5 (invalid `creel.yaml`): the refusal rests on `configengine`'s own error and states no next step, and it takes down the whole listing.
   - D5 degrades the same fault to untrusted at arrival.
   - Proposed change: name `lym config creel` as the way forward, or list the hubs with `trusted: false` plus a warning.

**Tool calls**
- Bash grep of `internal/state/state.go`, `zz:` in the creel tests and `lymtemp.ProcessRoot`: my context was truncated before `UpdateJSON` and lacked the test uses.
- Read of `internal/fabriccli/shortname.go`: needed to see whether a shortname can change, which my context did not show.
- Bash read of the board upsert help and `board.yaml` labels: needed to check that card 10's note payload can succeed, which my context lacked.
