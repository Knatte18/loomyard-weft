```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then checked each acceptance criterion against the decisions it touches.

**Pairs checked**
- D3 dobby-tick start × D6 presence: held. Ticks start the relay only while presence is `slack`, and `stop` sets `terminal` before it ends the relay.
- D3 `lym slack stop` × D6 refusal from a non-orch pane: held.
- D3 `start` (not refused) × D6 refusal of `away`, `back` and `stop`: held. `start` changes no presence.
- D3 anchor-change restart × D3 prime-bring-up start: **broken** (F1).
- D3 first creel read × D6 `away` waiting for it × D8 refused post retried: held.
- D3 re-exec × D5 posted-id record: held. The socket closes on exec, the daemon releases the read, and the on-disk record skips ids already posted.
- D3 single-instance lock × D4 cursor: held.
- D2 `setup` (channel or operator change) × D4 persisted cursor: **broken** (F2).
- D2 incomplete config × D3 refusals and start conditions: held.
- D4 Slack texts `away`/`back`/`status` × D6 verbs and notices: held.
- D4 presence read at each poll × D6 presence file: held.
- D4 `@&lt;slug&gt;` routing × D5 `(answer: @&lt;slug&gt; …)` hint: held, a beam program's post included.
- D4 pause answer × D8 pause clearing the wait start: held.
- D4 poll and 429 backoff × D5 retry "with the poll's backoff": held, one relay and one backoff state.
- D5 posting any program mail × D8 dobby reminders to SLACK: held. Mail waits while the relay is down, and reminders past 24 hours are skipped.
- D5 posting regardless of presence × D7 orch posting only replies while `terminal`: held, the orch's own discipline.
- D6 `[slack presence]` notices × D7 not answering them: held.
- D6 resume-pointer line × D6 presence file: held.
- D8 wait start in the state record × D3 anchor change ending the prime dobby: held. Only the in-memory last-reminder time is lost, which D8's bound states.
- D9 orch command index × D6 orch-pane permission: held.
- Constraints rate-limit line × D4 floor and D5 posts: held. The line counts history calls only, and `chat.postMessage` is a separate method.

**Acceptance criteria against decisions**
- D3 "the relay is live after a prime anchor change with presence terminal, whether the change lands or fails": broken (F1).
- All other criteria: no conflict between decisions found.

**Findings**

F1 — MEDIUM — consistency — decision-record.md D3, third bullet, and its acceptance criterion
- **Summary:** D3's restart of the relay on a prime anchor change rides on the dobby restart, so it fails exactly where the acceptance criterion says the relay is live.
- **Evidence, restart is conditional:** `startDobbies` returns nil when `ended` is empty (internal/fabriccli/anchor.go:472-476), but `EndSparing` over `runproc.HubOwner` ends the relay whether or not a prime dobby was live (anchordeps.go:248-252).
- **Evidence, failed change starts nothing:** `fail` starts no dobby when the change record stands or cannot be read (anchor.go:350-362).
- **Evidence, landed change already covered:** on a landed change, `EnsureDobbies` goes through `lym beam start` (anchordeps.go:264-269, `startDobbiesThroughBeamStart`), which is the prime bring-up D3 already lists as a start. On that path the separate anchor hook is redundant.
- **Consequence:** with presence `terminal` and a failed change whose record stands, or with no prime dobby live, the relay stays down. D3 itself says the change ends it.
- **Fix:** state the anchor-change start as unconditional after the change ends, or keep the hook conditional and narrow D3's text and its criterion to the paths that start dobbies.

F2 — MEDIUM — design — decision-record.md D2, third bullet, × D4, first bullet
- **Summary:** the D4 cursor is one persisted `ts` with no channel attached, and D2's `lym slack setup` can repoint the channel without touching it.
- **Consequences:**
  - The relay keeps reading the new channel from the old channel's cursor.
  - It forwards that channel's history posted after the old `ts`.
  - It also skips the first-start rule ("forwards no history").
- **Unstated:** neither decision says whether a running relay re-reads the channel and operator id after `setup`, or keeps the old ones until it restarts.
- **Evidence:** decision-record.md:40 (setup writes the keys) and :65-66 (cursor and first-start rule).
- **Fix:** key the cursor by channel id, or have `setup` reset it, so a changed channel takes the first-start path. State whether `setup` restarts a live relay.

**Tool calls:** none. My context held every file this lens needed.
