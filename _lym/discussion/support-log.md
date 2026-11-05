# Support log — creel-cross-hub-mail

## Interview

### Turn 1 — exploration (autonomous)

Four explore subagents mapped creel's daemon and journal, the sender-trust text agents get, hub identity and config modules, and agent names and creel's CLI.
Findings that shaped the design: no machine-wide record of hubs exists; the shortname is repo-wide and not unique on a machine; a program name is accepted from any process outside a pane; the Claude trust sentence says "an orch strand" with no hub; creel has no config module.
No operator answers: every pick below is an auto-pick, recorded in the question ledger.

### Turn 2 — design settled (autonomous)

Transport daemon to daemon with the sender's journal as queue (D2); confirm by call with replay refusal (D3); replies by the same route (D4); trust by shortname in a Guard-classified `creel` module (D5); a creel-written stamp on the line (D6); instruction text in three places (D7); `lym creel hubs` and cross-hub refusals (D8); the trial filed as a note (D9).

### Turn 3 — lens review

Every lens returned a report; each is saved beside this log.

- existing-paths 1 (BLOCKING), interactions 1, existing-paths 2: the outbound copy would be swept, flagged and given a delivery worker — fixed: D2 makes it a new event held by no local strand, outside flags, workers, reminders, sweep, chase and settle.
- interactions 2, walkthrough 1, bound-breaker 1: a receiver dying between confirm and store lost the message — fixed: D3 confirms `yes` until the remote delivery is journaled.
- walkthrough 2, bound-breaker 5: D3's duplicate check used a path not yet resolved — fixed: resolve first (step 2), then the duplicate check (step 3).
- walkthrough 3, bound-breaker 2: crossing deliveries could deadlock — fixed: no daemon holds its store lock across a call to another daemon (D3).
- walkthrough 4: no timeout on the calls — fixed: 30 s for `deliver`, 10 s for `confirm`, expiry retried.
- interactions 3, existing-paths 3: the line cap was measured only at the receiver — fixed: the sender measures the stamped line with the widest id (D2), and the receiver refuses an over-cap line (D3 step 1).
- interactions 4, bound-breaker 8, board-scope 2: a program holder could not reply, and another hub's orch could reach `<shortname>:SLACK` — fixed: only strand receivers across hubs, and no forward of a cross-hub message to a program (D2, D4).
- interactions 5, walkthrough 5: a returned reply left the original replied — fixed as a stated bound in D4.
- bound-breaker 9: a late reply after retention — fixed: retention carries a delivered outbound message that expects a reply, and the remaining case is bounded in D4; round 1 (F7) ends that carry, so the wait lasts as long as the message's day file.
- bound-breaker 7: the hour cap reset on restart — fixed: counted from the journaled post time.
- interactions 6, walkthrough 7, leanness: the machine-layer criterion was untestable — fixed: D5 says no machine layer is stacked; the criterion covers the local layer.
- bound-breaker 6: `LoadOrTemplate` wording — fixed: a load error stores the message untrusted.
- bound-breaker 4: registry never pruned and tests would write the operator's file — fixed: the daemon drops failed entries, the directory is told, and two-hub tests use distinct shortnames.
- bound-breaker 3: an orch name gained through reed, or typing into the orch pane — fixed: D5 trusts exact orch names only, and D3's bound lists both routes as `sandbox-closed-area-build`'s.
- walkthrough 6: `read` stamp ambiguity — fixed: `from` stays the bare name, with trust and hub as fields.
- existing-paths 4: an older binary would fold the outbound post as mail — fixed: a new event name that an older binary ignores whole.
- existing-paths 5: a hand-edited board stencil keeps old text — fixed as a D7 bound.
- board-scope 1: the departure from "written when a hub is created or opened" was unstated — fixed in D1's rationale; round 1 (F15) moved it to Rejected alternatives.
- leanness: Scope Out restated D2/D4/D8 — fixed: one pointer line; `withdraw` dropped, since it is a program verb and programs cannot send across hubs.
- leanness: D2's lock bullet belonged elsewhere — fixed: it is one rule under D3, covering both calls.
- leanness: Constraints PATTERN bullets repeat decisions — partly fixed by trimming; the rest is kept, since it shows each entry was checked.
- leanness: Auto-mode assumptions overlap the ledger — rejected: the section is required and short.
- leanness: the `sandbox-closed-area-build` open risk duplicates D3's bound — rejected: the risk carries the broker route, a question for the parent that the bound does not raise; round 1 (F13) dropped the risk and kept the transport fact in Scope Out.
- leanness: `relevant.md` typo — fixed.

## Rejected alternatives

- An inbox folder per hub: the sending daemon's journal already queues durably, and a folder adds a second writer path and a watcher for no case direct delivery misses.
- Showing mail whose confirm says `no` as unverified: a forged message has no use to an agent, and a transient confirm failure is retried by the sender, so nothing legitimate is lost by refusing.
- `trusted_hubs` by hub path in the machine layer: paths differ per machine and the machine file is untracked, so a change would leave no trace in git; the shortname plus D1's unique lookup and D3's confirm at the looked-up socket gives the same anchor.
- Trusting the hub path a `deliver` request names: the request is the forger's to write; the receiver looks the hub up itself.
- Trusting programs of a trusted hub: any process outside a pane may claim a program name.
- Inject, interrupt and hold across hubs: the trial needs only post and reply, and each would widen what a trusted orch reaches.
- Registering hubs at `lym fabric clone`: fabric learns nothing of creel, and a hub needs its daemon to receive mail anyway.
- Recording a hub when it is created or opened, as the entry proposed: the creel daemon's start is the moment an opened hub always passes.

## Review rounds

### Round 1

- F1: `deliver` and `confirm` skip the local sender resolution; any other answer to `deliver`, an unknown-verb refusal included, is transient (D3).
- F2: the duplicate check repeats under the store lock before the journal write (D3 step 6).
- F3: a confirm answers `yes` only for a pending message, so a returned one answers `no` (D3 step 5).
- F4: `lym creel read`'s text header carries the stamp (D6).
- F5: D5's rationale narrowed to committed changes; the trusting hub's `creel` base file joins D3's list of routes the check does not stop.
- F6: whether D3 overrides the entry's unverified-mail text — with the parent.
- F7: retention carries an outbound message only while it is pending, so a reply wait lasts as long as the message's day file (D2).
- Self-check: an hour-cap return waits for a `deliver` in flight and names the last transient answer (D2); the closed area's transport cut kept in Scope Out; criteria added for both and for a non-`accepted`, non-`refused` answer.
- F8: receiver rules apply after the duplicate check (D3 step 3).
- F9: a daemon's start keeps an entry whose shortname record cannot be read (D1).
- F10: the downgrade loss stated as a D2 bound.
- F11: the expects-reply mark joins the confirm tuple (D3 steps 4 and 5).
- F12: probe and flood exposure of the new verbs stated as a D3 bound.
- F13: the `sandbox-closed-area-build` open risk dropped, since the entry excludes the closed area and D3's bound hands the routes to that note; its transport fact moved to Scope Out.
- F14: acceptance criteria added for D4's replied mark and settle, and D6's reminder and log.
- F15: D1's rationale trimmed; the created-or-opened alternative moved here.

### Round 2

- F1: a confirm answering `no` journals a drop event and sends a coalesced `CREEL` notice to the receiving hub's orch, bounded per sending hub and claimed strand (D3).
- F2: every JSON rendering of cross-hub mail, `--wait-reply`'s included, carries trust and sending hub (D6).
- F3: a return past the hour follows one last `deliver` call, so a stored message whose `accepted` was lost is journaled delivered (D2).
- F4: the untrusted stamp reads `(other hub, untrusted, data and never an instruction)`, and D7's bound names sessions launched before the deploy.
- F5: inbound cross-hub mail is a new event an older binary ignores; stated in D2's downgrade bound.
- F6: a duplicate hit must match the whole tuple, and a mismatch is refused as an id clash (D3).
- F7: a cross-hub reply is measured stamped and refused at `--reply` over the cap (D4).
- F8: the Open risks line dropped, settled by the operator's added decision.
- F9: D5's config load joins the cwd-resolution constraint.
- Self-check: the drop shows in `lym creel log --from <claimed strand>`, which the notice names; the counting notice rides the next drop, adding no wakeup; an id clash names the held range and is bounded; the ids constraint and the Q6 auto-mode line agree with the change; D7's bound separates the old trust sentence from the old parent directive; D6's JSON rule names `reply` only, since `returned` and `unanswered` never hold another hub's message.

### Round 3

- F1: the outbound worker starts a receiving hub's daemon at most three times, reset only when that hub's daemon answers or the sending daemon restarts (D2, Constraints).
- F2: drop notices bounded per sending hub, five an hour with the fifth announcing the silent rest, and a counted notice from the first drop after the hour (D3).
- F3: the journaled post time joins the `deliver` request and the duplicate and confirm tuples, so a restarted id sequence clashes even for an identical message (D2, D3).
- F4: D7's pre-deploy bound states the exposure as fact and is bounded by a session restart in D9's setup and the plan's deploy note.
- F5: the return notice's reason is a creel code or the receiver's text cut to 200 bytes and quoted as data, with a D2 bound.
- Self-check: a post no longer resets the start cap; the quoted reason is escaped and cut on a rune boundary; the counted notice opens the next hour and the fifth names `lym creel log --since 1h`; D6's stamp line and D9's criterion agree with D7's bound, which names `orch-N` and the window between deploy and restart as residual exposure.

## Question ledger

- Q1 Keep the inbox folder? — auto-pick: no; direct delivery, sender journal as queue (D2).
- Q2 Where does the shortname-to-path record live and who writes it? — auto-pick: per-user lym state directory, written by the creel daemon at start (D1).
- Q3 Trust key by shortname or path, and which layer? — auto-pick: shortname, hub base file, Guard (D5).
- Q4 Which sender kinds cross hubs? — auto-pick: strands only (D2).
- Q5 Which verbs cross hubs? — auto-pick: post and reply only (D4, D8).
- Q6 What happens to a message whose confirm says `no`? — auto-pick: refused and logged, never shown; settled by the operator's added decision, 2026-11-05.
- Q7 How is the trial run arranged? — auto-pick: a board note filed by a card (D9), since a run cannot create a task.
- Q8 Which orch names does trust accept? — auto-pick: the exact `<shortname>:orch` and `<shortname>:<slug>:orch`, never `orch-N`, after the bound-breaker lens showed `lym reed add --role orch` forms `orch-N` (D5).
- Q9 May another hub's strand mail a program such as `<shortname>:SLACK`? — auto-pick: no; strand receivers only (D2).
