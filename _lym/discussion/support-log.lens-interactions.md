```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, and the acceptance criteria against the decisions they touch.

**Findings**

1. **MEDIUM** (D2, plus creel's sweep, delivery worker and flags): D2 never takes the sending hub's outbound copy out of the local machinery that acts on unread mail.
   - Evidence: by `internal/creelengine/doc.go`, a sweep runs while a holder has unread mail, and a holder the strand list misses twice counts as gone, so its message steps back to the sender. The Scheduler runs one worker per receiver with unread mail, and flags are kept per holder.
   - D2 journals the outbound message with its receiver set to `lyt:orch`, which is not one of hub A's strands. It does not say whether that copy is read, unread or a separate kind.
   - With D2 in force, A's sweep would return the message to its own sender within about 60 seconds, possibly while it is still pending or already delivered. A's delivery worker would also probe a strand that does not exist.
   - Fix: D2 should state that an outbound message is outside the pool's holder logic (sweep, workers, flags, chase), and say which state it is in after remote delivery.

2. **MEDIUM** (D3 against D2's retry): a crash in the confirm window loses a legitimate message and reports it to the sender as refused.
   - Step 5 journals `confirmed` and answers `yes`, and step 6 stores the message afterwards. Suppose the receiving daemon dies, or its journal write fails, after `yes` but before it stores.
   - D2 then retries. Step 2 finds no duplicate, so step 4 asks for confirmation again, and step 5 answers `no` because the message is already confirmed. Step 6 refuses, and D2 returns the message undelivered at once with a `CREEL` notice.
   - D2's bound ("exactly once in the receiver's pool") does not hold: the message arrives zero times.
   - Fix options: let a repeat confirm from the same receiving hub answer `yes` until the sender journals the remote delivery, or make the receiver journal a pending arrival before it confirms.

3. **MEDIUM** (D6 against D2): the 8 KiB line cap is checked at the wrong hub.
   - D6 says the cap measures the stamped line. But D2 accepts and answers the post at the sending daemon, which does not know the receiver's id width or which stamp the receiver will add.
   - A body that passes the sender's check can go over the cap on the receiver. The record does not say who refuses it or what happens next: a refused delivery, a return, or a line past the hook-field budget.
   - Fix: the sender measures with the longer stamp and a margin for the id, or D3 lists an over-cap line as a refusal reason that returns the message at once.

4. **LOW** (D2 against D4): a program that holds a message from another hub cannot reply to it.
   - D2 refuses cross-hub sends from programs. D4 routes a reply by the current holder across the border, and the receiver rules allow a program that has read to be a holder, such as `lyt:SLACK`.
   - So D4's "a reply by its current holder" is refused by D2 when the holder is a program. One of the two should name the exception.

5. **LOW** (D4 against D2's return path): a reply that comes back undelivered leaves the original marked replied.
   - D4 marks the original replied as soon as the reply is journaled, which stops its chase. If D2 later returns that reply undelivered, the original stays marked replied and is never chased again.
   - D4's bound covers only an unanswered settle and a gone holder, not a returned reply.

6. **LOW** (Acceptance criterion D5 against D5): the criterion "`trusted_hubs` is refused in a machine layer file" will not happen as written.
   - configengine stacks the machine layer only for a hub-wide module that has a `CategoryMachine` key (`internal/configengine/layers.go`). The `creel` module has none, so a `creel.machine.yaml` is never read, rather than refused.
   - Only the local-layer half of the criterion holds. The machine-file half should become "is not stacked", or be dropped.

**Pairs checked**
- D1 and D3: shortname lookup and the refusal on several hubs — held.
- D1 and D2: lookup at the post, then the stored path at delivery — held.
- D1 and D8: `lym creel hubs` reads the record — held.
- D2 and D3: confirm on a second connection while the deliver call is in flight — held.
- D2 and D3: retry after a lost `accepted`, answered by the duplicate rule — held.
- D2 and D3: crash between the confirm and the store — broken (finding 2).
- D2 and the existing sweep, worker and flags over the outbound copy — broken (finding 1).
- D2 and D4: reply from a program holder — broken (finding 4).
- D2 and D4: returned reply against the replied mark — broken (finding 5).
- D2 and D6: line cap at the sender against the stamp at the receiver — broken (finding 3).
- D2 and the retention constraint: outbound carried until delivered — held.
- D3 and D4: reply-to carried in the deliver call and the confirm — held.
- D3 and D4: receiver rules refusing a removed original sender, which returns the reply with a notice — held.
- D3 and retention: replay after either side's retention is refused by confirm — held.
- D3 step 1 and D2: own shortname is local — held.
- D4 and D8: forward is refused in D4, and the other cross-hub verbs in D8 — held.
- D5 and D6: trust journaled at arrival and rendered from the stored value — held.
- D5 and D3: the off-Linux bound, on the same machine and so the same OS — held.
- D5 and D7: own orch strands plus the trusted stamp — held.
- D6 and pane-mail delivery: cross-hub mail is reminded only — held.
- D5 and its acceptance criterion: the machine-layer half — broken (finding 6).
- D9 and Scope: the trial as a note — held.
- PATTERN change and D2/D5: strands only, authority only for a trusted hub's orch — held.

**Tool calls**
- Grepped `internal/configengine` for machine-layer refusal, to check D5's acceptance criterion. My context did not show how the machine layer treats a module with no machine key.
