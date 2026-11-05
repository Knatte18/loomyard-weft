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
- bound-breaker 9: a late reply after retention — fixed: retention carries a delivered outbound message that expects a reply, and the remaining case is bounded in D4.
- bound-breaker 7: the hour cap reset on restart — fixed: counted from the journaled post time.
- interactions 6, walkthrough 7, leanness: the machine-layer criterion was untestable — fixed: D5 says no machine layer is stacked; the criterion covers the local layer.
- bound-breaker 6: `LoadOrTemplate` wording — fixed: a load error stores the message untrusted.
- bound-breaker 4: registry never pruned and tests would write the operator's file — fixed: the daemon drops failed entries, the directory is told, and two-hub tests use distinct shortnames.
- bound-breaker 3: an orch name gained through reed, or typing into the orch pane — fixed: D5 trusts exact orch names only, and D3's bound lists both routes as `sandbox-closed-area-build`'s.
- walkthrough 6: `read` stamp ambiguity — fixed: `from` stays the bare name, with trust and hub as fields.
- existing-paths 4: an older binary would fold the outbound post as mail — fixed: a new event name that an older binary ignores whole.
- existing-paths 5: a hand-edited board stencil keeps old text — fixed as a D7 bound.
- board-scope 1: the departure from "written when a hub is created or opened" was unstated — fixed in D1's rationale.
- leanness: Scope Out restated D2/D4/D8 — fixed: one pointer line; `withdraw` dropped, since it is a program verb and programs cannot send across hubs.
- leanness: D2's lock bullet belonged elsewhere — fixed: it is one rule under D3, covering both calls.
- leanness: Constraints PATTERN bullets repeat decisions — partly fixed by trimming; the rest is kept, since it shows each entry was checked.
- leanness: Auto-mode assumptions overlap the ledger — rejected: the section is required and short.
- leanness: the `sandbox-closed-area-build` open risk duplicates D3's bound — rejected: the risk carries the broker route, a question for the parent that the bound does not raise.
- leanness: `relevant.md` typo — fixed.

## Rejected alternatives

- An inbox folder per hub: the sending daemon's journal already queues durably, and a folder adds a second writer path and a watcher for no case direct delivery misses.
- Showing mail whose confirm says `no` as unverified: a forged message has no use to an agent, and a transient confirm failure is retried by the sender, so nothing legitimate is lost by refusing.
- `trusted_hubs` by hub path in the machine layer: paths differ per machine and the machine file is untracked, so a change would leave no trace in git; the shortname plus D1's unique lookup and D3's confirm at the looked-up socket gives the same anchor.
- Trusting the hub path a `deliver` request names: the request is the forger's to write; the receiver looks the hub up itself.
- Trusting programs of a trusted hub: any process outside a pane may claim a program name.
- Inject, interrupt and hold across hubs: the trial needs only post and reply, and each would widen what a trusted orch reaches.
- Registering hubs at `lym fabric clone`: fabric learns nothing of creel, and a hub needs its daemon to receive mail anyway.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Keep the inbox folder? — auto-pick: no; direct delivery, sender journal as queue (D2).
- Q2 Where does the shortname-to-path record live and who writes it? — auto-pick: per-user lym state directory, written by the creel daemon at start (D1).
- Q3 Trust key by shortname or path, and which layer? — auto-pick: shortname, hub base file, Guard (D5).
- Q4 Which sender kinds cross hubs? — auto-pick: strands only (D2).
- Q5 Which verbs cross hubs? — auto-pick: post and reply only (D4, D8).
- Q6 What happens to a message whose confirm says `no`? — auto-pick: refused and logged, never shown; raised under Open risks since the entry proposed showing it as unverified.
- Q7 How is the trial run arranged? — auto-pick: a board note filed by a card (D9), since a run cannot create a task.
- Q8 Which orch names does trust accept? — auto-pick: the exact `<shortname>:orch` and `<shortname>:<slug>:orch`, never `orch-N`, after the bound-breaker lens showed `lym reed add --role orch` forms `orch-N` (D5).
- Q9 May another hub's strand mail a program such as `<shortname>:SLACK`? — auto-pick: no; strand receivers only (D2).
