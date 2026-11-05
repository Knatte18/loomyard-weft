---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2 and D3 (deliver and confirm requests)
    summary: deliver and confirm reach the daemon through its local sender resolution, which refuses a peer in no pane whenever a live strand's pane process is unknown, and the record neither exempts the two verbs nor classifies that refusal among accepted, refused and transient
    fix: In D3, add a bullet stating that the deliver and confirm requests skip the local sender resolution of internal/creelengine/server.go (resolve before dispatch), since the confirm alone settles who sent; and that any answer to deliver other than accepted or refused, an older binary's unknown-verb refusal included, counts as a transient refusal the sending daemon retries under D2. Add a D3 acceptance bullet - a deliver while a live strand of the receiving hub has no known pane process is accepted, not refused.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 steps 3 and 6, and the store-lock paragraph
    summary: the duplicate check of step 3 and the store of step 6 sit on either side of the confirm call with no lock held between them, so two overlapping deliveries of one tuple, a retry after a timed-out deliver included, both pass step 3 and both store, breaking D2's exactly-once bound
    fix: In D3 step 6, state that the receiving daemon repeats the duplicate check (sending hub path and sending id) under its store lock before it journals, and answers accepted with the existing id when it finds one. Extend the D3 acceptance bullet with - two overlapping deliveries of one message store it once and answer the same id.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 step 5 against D2's return rules
    summary: the sending daemon answers yes to a confirm while no remote delivery is journaled, even after D2 returned the message undelivered by a refused answer or the hour cap, so a deliver in flight at the return is stored and acted on after the sender was told it failed
    fix: In D3 step 5, answer yes only for an outbound message that is still pending - no remote delivery and no return journaled. Add to the D3 acceptance bullet - a confirm for a message already returned undelivered answers no.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 second bullet (lym creel read)
    summary: lym creel read's text output prints a header of the bare sender (internal/creelcli/cli.go writeMessagesText), and a pane-mail strand such as every Codex strand reads cross-hub mail only through it, so trusted and untrusted mail look alike exactly where D7's instructions tell the agent to read the stamp
    fix: In D6, state that the text header of lym creel read carries the same stamp after the quoted sender as the hook line, while the JSON keeps from as the bare name with trust and sending hub as their own fields. Add to the D6 acceptance bullet - the text output of lym creel read shows the stamp.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 rationale and D3's same-user bound
    summary: D5 reads trusted_hubs from the base file on disk at each arrival, so an uncommitted edit by any process of the user, a sandbox agent included, grants trust at once; D5's rationale says every change is kept in git, and D3's list of routes the check does not stop omits the trusting hub's config file
    fix: In D5's rationale, drop the claim that committing the list keeps every change in git, or narrow it to committed changes. In D3's bound listing what the check does not stop, add a process that writes the trusting hub's creel base file, beside the journal files and the machine record, as one more route that is sandbox-closed-area-build's to close.
  - id: F6
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks first bullet; D3 steps 4 and 6
    summary: the board entry's operator-written section Verifying the sender says a message the confirm does not verify shows as unverified, and that a sending daemon that is down is started for the confirm; D3 drops such a message and starts no daemon for the confirm, overriding the operator's own text, which only the parent can accept
    fix: Parent decides whether D3's stricter reading stands - a confirm answering no drops the message, and the confirm starts no daemon - or the operator's text in the entry, under which such mail shows as unverified data. Record the answer with lym loom decision add; the writer then drops the Open risks line.
  - id: F7
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D2 last bullet (retention) and D4 bound
    summary: retention carries an outbound message while it expects a reply and has none, and nothing on the sending side ever ends that state when the receiver settles it unanswered, its holder is gone, or it returned undelivered, so such a message is carried at every retention forever
    fix: In D2's retention bullet, carry an outbound message that expects a reply only while it is pending or delivered with no reply and younger than the journal's seven-day retention counted from its post time; a returned message is not carried. Keep D4's bound that a reply after the sending side dropped its message is refused.
  - id: F8
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 steps 1 to 3
    summary: the receiver rules of step 1 run before the duplicate check of step 3, so a retry after a lost accepted, for a receiver strand that has since ended, is refused and the sender is told a delivered message returned undelivered
    fix: In D3, apply the receiver rules after the duplicate check - step 1 keeps the shape checks (sender a strand name, not of the own shortname, line cap), step 2 resolves the sending hub, step 3 answers a held message's id, and only then are the receiver rules applied.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 third and fourth bullets and Bound
    summary: a daemon's start drops every entry whose shortname read fails, and fabricengine.ReadShortname answers not-found on any read error, so a transiently unreadable live hub is dropped and stays unaddressable until its own daemon restarts, against D1's bound that a hub is addressable once its daemon has started
    fix: In D1, have the daemon's start drop only entries whose hub path no longer exists or whose shortname record reads a different valid shortname, and keep an entry whose record could not be read; the lookup still filters by the current read.
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 outbound-event bullet
    summary: an older binary ignores the outbound event in the fold, but its day-change retention carries only the kinds it knows and deletes the old files, so after a downgrade pending outbound messages are lost without a return or notice; the record states only the fold
    fix: In D2's outbound-event bullet, add a bound - a downgraded daemon's retention drops pending and reply-awaiting outbound messages older than seven days with no return and no notice.
  - id: F11
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D3 step 4 (confirm fields)
    summary: the confirm tuple omits the expects-reply mark the deliver carries, so a deliver confirmed yes may carry a mark the sender never set, and the duplicate rule then keeps that copy over the real one
    fix: In D3 steps 4 and 5, add the expects-reply mark to the confirm tuple and to the sending daemon's match.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D3 bounds
    summary: deliver and confirm are new socket verbs any process of the user can send, and D3 bounds only the forged sender; it states neither what a confirm answers a probing caller nor that each deliver costs a machine-record lookup and a confirm call of up to 10 seconds to another hub's daemon
    fix: Add one Bound line to D3 - a confirm answers only yes or no to a full tuple that includes the body hash, so it tells a caller nothing it could not learn by reading the sending hub's journal; a deliver flood costs the receiver one record lookup and one confirm per request, with no cap, the same exposure any process of the user has to any verb.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks second bullet
    summary: the risk about sandbox-closed-area-build raises a question the entry already settles - its Not in this round excludes how this fits a closed area, Scope Out repeats it, the closed area is a note and not a run, and D3's bound already hands those routes to that note
    fix: Drop the second Open risks bullet.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Acceptance criteria, D4 and D6 bullets
    summary: behaviour D4 and D6 state has no criterion - the replier-side replied mark that stops the chase, the settle that sends no notice to the other hub, lym creel log showing the sending hub's path and id, and the reminder's stamp
    fix: Extend the D4 acceptance bullet with - the replier's message is marked replied and its chase stops when the reply is journaled, and a settle of a cross-hub message sends no notice to the other hub. Extend the D6 bullet with - the unread reminder shows the stamp after the sender, and lym creel log shows the sending hub's path and id on the message's entry.
  - id: F15
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D1 Rationale
    summary: D1's rationale argues against the entry's proposal to record hubs when a hub is created or opened and against fabric's clone, which re-litigates the rejected alternative support-log.md already holds
    fix: In D1's Rationale, keep the first clause (no machine-wide list exists, and the creel daemon is the one process every hub starts that mail needs) and delete from the entry proposed to the end of the sentence.
---

# Round 1 review — creel-cross-hub-mail discussion

The record is sound in shape: every decision has acceptance coverage, the attack-surface bounds are mostly stated, D9 files a note and no decision edits this task's own entry or files a task.
The kept findings are mechanism gaps in the delivery and confirm protocol, one display gap that hides the stamp on the path pane-mail strands use, one omitted same-user route to trust, and one departure from the operator's own text that only the parent can accept.

### [MEDIUM] deliver and confirm meet the local sender resolution
**Location:** decision-record.md D2, D3
**Issue:** `internal/creelengine/server.go` resolves every request before dispatch, and `resolveSender` in `internal/creelengine/identity.go` refuses a peer in no pane when a live strand's pane process is unknown. The other daemon is such a peer. The record leaves this refusal unclassified, and read as `refused` it returns a message at once. Found by three lenses and the handler.

### [MEDIUM] Overlapping deliveries are stored twice
**Location:** decision-record.md D3 steps 3 and 6
**Issue:** No lock spans the confirm call, so the duplicate check and the store are not atomic. A retry after a timed-out `deliver` overlaps the first handler and both store. Found by interactions and bound-breaker.

### [MEDIUM] A returned message still confirms yes
**Location:** decision-record.md D3 step 5
**Issue:** The confirm's `yes` rests only on the absence of a remote delivery. A message D2 already returned undelivered still confirms, so the receiver acts on what the sender was told failed. Found by three lenses.

### [MEDIUM] lym creel read text shows no stamp
**Location:** decision-record.md D6
**Issue:** `writeMessagesText` in `internal/creelcli/cli.go` prints `#<id> from "<sender>"` with nothing else. A pane-mail strand gets only a reminder and reads the body here, so D7's instruction to read the stamp has nothing to read on that path.

### [MEDIUM] An uncommitted config edit grants trust
**Location:** decision-record.md D5, D3 bound
**Issue:** `LoadOrTemplate` reads files on disk. A sandbox process writing loomyard's `creel` base file makes `lyt:orch` trusted at the next arrival, which is the one-way trust the Goal protects. The bound list omits this route, and D5's rationale says the opposite.

### [MEDIUM] D3 overrides the operator's text on unverified mail
**Location:** decision-record.md Open risks first bullet, D3
**Issue:** The entry's operator-written section says unverified mail shows as unverified, and that a down sending daemon is started for the confirm. D3 drops such mail and starts no daemon for the confirm. The writer raised the first part, rightly, since it overrides the operator's own design and not a proposal. The second part is the same departure and goes to the parent with it. No `Decision for creel-cross-hub-mail:` note exists on the board.

### [LOW] Reply-awaiting outbound messages are carried forever
**Location:** decision-record.md D2 retention bullet
**Issue:** The sending side never ends the wait, so the carry rule holds such a message forever after a settle on the other side, a gone holder or a return.

### [LOW] Receiver rules before the duplicate check
**Location:** decision-record.md D3 steps 1–3
**Issue:** A retry for a receiver that ended since the store is refused, and a delivered message is reported returned.

### [LOW] D1 prunes a live hub on a read error
**Location:** decision-record.md D1
**Issue:** `fabricengine.ReadShortname` returns not-found on any read error, so another daemon's start can drop a live hub until that hub's daemon restarts.

### [LOW] Downgrade loses outbound messages
**Location:** decision-record.md D2
**Issue:** An older binary's retention deletes files holding events it does not carry. The record states only the fold.

### [LOW] Confirm tuple omits the expects-reply mark
**Location:** decision-record.md D3 step 4
**Issue:** The mark travels in `deliver` but the confirm does not check it.

### [LOW] Probe and flood exposure of the new verbs unstated
**Location:** decision-record.md D3
**Issue:** The focus asked for the bounds of a `confirm` probe and a `deliver` flood. Neither does real harm beyond what any process of the user already has, but the attack-surface rule wants the Decision to state it.

### [LOW] Open risk on sandbox-closed-area-build is already settled
**Location:** decision-record.md Open risks second bullet
**Issue:** The entry excludes the closed area from this round, `sandbox-closed-area-build` is a note and not a run, and D3's bound covers it. This is not a `decision` question.

### [LOW] Acceptance gaps in D4 and D6
**Location:** decision-record.md Acceptance criteria
**Issue:** Four stated behaviours have no criterion.

### [NIT] D1 rationale re-litigates a rejected alternative
**Location:** decision-record.md D1 Rationale
**Issue:** The argument against recording at clone or open belongs only in `support-log.md`'s Rejected alternatives, which already holds it. The plan writer needs no part of it.

## Rejected

- No finding against D9 or any board edit: D9 files a note through a card, and no Decision changes this task's entry or files or promotes a task.
- No finding on D5's trust bound for a forged trusted orch: forging trust only gains authority over the sandbox hubs, which the sandbox agents already reach, and D5 bounds the grant to what the hub's own orch carries.
- No finding on `agentname.Parse` or on the outbound event in the fold: both rest on code read (`internal/agentname/agentname.go` Parse checks grammar only; `internal/creelengine/doc.go` says a reader ignores unknown events).
- No PATTERN-lym-files finding on `hubs.json`: the record belongs to no hub and follows the per-user precedent of `standalonestate`, which the pattern's background names.

Process note: the handler's own reasoning formed early judgments before the forks were spawned, which the explore step forbids. The fork prompts carried only the boilerplate and the lens text.
