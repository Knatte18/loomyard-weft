---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 bullet 2, D6 bullet 2, Acceptance criteria D3 bullet
    summary: D3's premise that a relay started detached in a new session sits outside every pane's process tree is false while its starter lives, so its SLACK program claim is refused when a non-orch strand pane starts it
    fix: In D3 bullet 2, replace the claim with the real mechanism, for example that the starter spawns the relay through a double fork (an intermediate child that starts the relay and exits at once, so the relay is reparented out of the pane tree before its first read), or that the relay waits until its parent pid is no longer the starter before its first creel read. State the bound, that a relay is never in a pane's tree at its first read. In D6 bullet 2, keep the wait for the first read, which then holds from any pane. In the D3 acceptance criterion, replace "the relay runs in no pane's process tree, so creel takes its SLACK claim when lym slack start runs from a strand's pane" with a criterion that `lym slack away` and `lym slack start` run from a non-orch strand's pane bring up a relay whose first read is accepted as `<shortname>:SLACK`.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D4 bullets 1-2
    summary: A first start with no cursor starts from the current time but persists nothing until a message is handled, so a relay that dies and is restarted before any message loses every message written between its death and its restart
    fix: In D4 bullet 1, say that a first start with no cursor writes the cursor as the current Slack time (the ts of the newest channel message, or now) before its first poll, so a later restart reads from there. Add to the D4 acceptance criterion on restarts that a message written while a relay that never handled a message is down is forwarded once after its restart.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D3
    summary: An anchor change of the prime ends the relay, because fabriccli EndDobbies ends every HubOwner process except push and sync and EnsureDobbies restarts only dobbies; with presence terminal the relay then stays down, and the record neither states nor bounds this
    fix: In D3, decide one of the two and state it. Either the anchor change spares the relay's role, with `EndDobbies` called with the relay role beside `runproc.RolePush` and `runproc.RoleSync` in `internal/fabriccli/anchor.go`. Or the anchor change's restart step starts the relay again whenever the hub's slack config is complete, beside `EnsureDobbies`. Add an acceptance criterion that the relay is live after a prime anchor change with presence `terminal`.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 bullets 3-4, Acceptance criteria D3 bullet
    summary: lym slack stop is silently undone while presence is slack, because the prime dobby starts a relay that is down at its next tick, and the record says neither what stop does in that state nor what bounds it
    fix: In D3 bullet 4, decide what `lym slack stop` does while presence is `slack`. Recommended is that it sets presence to `terminal` first, with D6's notices, and then ends the relay, so the dobby restart does not fire. The alternative is that it refuses, naming `lym slack back`. State the bound, and add the chosen behavior to the D3 acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5 Bound line
    summary: The bound that creel's seven-day retention bounds how long mail waits for a relay that stays down is false, since retention carries every unread message and a program's read forward, so SLACK mail waits forever and a relay back after a long stop posts its whole stale backlog to the phone
    fix: In D5, drop the retention clause from the Bound line, with its evidence (`internal/creelengine/journal.go` `retain` carries unread messages and a program's read forward). Decide what the relay does with old mail. Recommended is that at a read it acknowledges, without posting, each message older than a stated age, such as 24 hours, and posts one line naming how many it skipped and `lym creel log --to <shortname>:SLACK`. State that bound, and add it to the D5 acceptance criteria.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 bullet 4, D4 bullet 3
    summary: Releasing a whole read when one post fails and reposting from its first message, with D4 resetting the wait to 5 seconds on every outbound post, makes a message Slack keeps refusing, or a backlog that hits Slack's posting limit partway, repost the earlier messages every cycle and never drain
    fix: In D5 bullet 4, keep a persisted record under `.lym/slack/` (through `internal/state`) of the creel ids already posted from the current read, skip them when the read is released and read again, and acknowledge once every id is posted or skipped. After a stated number of failed tries of one message with a non-429 error, post one line naming its id and sender and treat it as handled. In D4 bullet 3, reset the wait only on a post that succeeds. Restate the D5 acceptance criterion so a failure on the second message, then success, posts each message exactly once.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D8 bullet 2 and Bound line, Acceptance criteria D8 bullet
    summary: The wait start, the At of the history entry that made the row current, is ambiguous (the previous row's outcome entry or a goto entry) and is not rewritten by a pause or resume, so a run resumed at PR-Review after a long pause is reminded at once, against D8's bound that a resume restarts the clock
    fix: In D8 bullet 2, define the wait start as the time dobby first reads the child running at the row with no pause covering it. Keep it in dobby's state record (`.lym/beam/` state, tolerant JSON), and clear it when the child leaves the row, stops running or a pause covers it. Restate the Bound line to match, so a pause or resume restarts the clock and a restarted dobby keeps it. Change the D8 acceptance criterion's "counted from the history entry that made the row current" to that definition, and add a case where a child paused and resumed at PR-Review gets no reminder before a full interval after the resume.
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 bullets 2-3, D9
    summary: The new verbs lym slack away and lym slack back change where the operator's messages and reminders go, yet no Decision states who may run them or what bounds them, so any agent strand can run lym slack back and silence Slack mirroring and reminders while the operator is away
    fix: In D6, add a Bound line naming who may change presence and what that lets through. Recommended is that `away` and `back` refuse when run from a strand pane other than an orch's, through creel's sender resolution or the pane ancestry, naming the operator's terminal or the Slack texts as the way forward. The bound is then that only the operator, an orch or a process outside every pane changes presence. Add the refusal to the D6 acceptance criterion.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 Rationale, Constraints Slack rate-limit bullet
    summary: The Tier 3 history limit is per app per workspace and one app serves every hub (D1), so the 5-second floor keeps the machine inside the limit only for four hubs or fewer, which the record does not state
    fix: In the Constraints rate-limit bullet, state that the limit is shared by every hub's relay on the machine's one app, and that the 5-second floor fits four hubs at 48 requests a minute. In D4 bullet 3, say that a `429` doubles the floor for that relay, so more hubs degrade to slower polling. Alternatively, state a hub count the design is sized for.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 Bound line
    summary: The bound that a hold delays the mail up to the hold's hour is false, because the same owner renews a hold by holding again, so a renewing holder delays the operator's mail with no limit
    fix: In the D4 Bound line, replace "up to the hold's hour" with "for as long as the holder keeps renewing it, at most an hour past its last renewal".
  - id: F11
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D4 bullet 7
    summary: An @slug message posted to a beam orch whose beam is torn down before the orch reads it stays unread under the orch's name forever, since creel never sweeps program mail to a hub orch, and the operator is never told
    fix: In the D4 Bound line, state that a message routed to a beam torn down before its orch reads it stays unread in creel and is not answered in Slack, and that `lym creel log --to <shortname>:<slug>:orch` shows it. Alternatively, have the relay withdraw its unread mail to an orch whose beam left the board (creel's program-only `withdraw` verb) and answer in Slack that it was not delivered.
  - id: F12
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 bullet 2, D8 bullet 3
    summary: A beam dobby's reminder comes from the program <shortname>:<slug>:DOBBY, but D5 adds the @slug answer hint only to a beam orch's posts, so an operator who answers that reminder in Slack reaches the prime orch, not the beam's
    fix: In D5 bullet 2, add the `@<slug>` answer hint to every post whose stamped sender carries a beam slug, `<shortname>:<slug>:orch` or `<shortname>:<slug>:<PROGRAM>`, and change the matching D5 acceptance criterion to match.
  - id: F13
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D4 bullets 1-2 and 5, Constraints Slack facts
    summary: Slack facts the cursor and the text handling rest on are not recorded with a source, namely conversations.history's oldest exclusivity, its newest-first order with has_more pagination, Retry-After as a header, and Slack's rewriting of URLs and mentions into angle-bracket forms, which D4 contradicts by saying the rest of the text stays as the operator wrote it
    fix: In Constraints' Slack facts, add with their source (docs.slack.dev/reference/methods/conversations.history and docs.slack.dev/messaging/formatting-message-text) that `oldest` is exclusive unless `inclusive` is true, that messages come newest first with `has_more` and a `response_metadata.next_cursor` to page, and that a 429 carries a `Retry-After` header in seconds. Also add that Slack sends a URL as `<url>` or `<url|label>` and a mention as `<@U…>`. In D4 bullet 1, say the relay pages until `has_more` is false before handling oldest first. In D4 bullet 5, say what the relay does with the angle-bracket forms, for example that it turns `<url|label>` and `<url>` into the URL, and leaves the rest as Slack delivers it.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 Rationale, D6 bullet 3 vs Acceptance criteria D6 bullet
    summary: D5's rationale says every outbound path goes through creel's journal although the relay answers in Slack directly (unknown slug, long body, paused hub, presence), and the D6 criterion asks the CLI verbs back and status to answer in Slack, which D6 does not decide
    fix: In the D5 Rationale, narrow the claim to "every agent's and program's outbound message goes through creel's journal; only the relay's own answers to the operator's Slack messages are posted directly". In the D6 acceptance criterion, split it so `lym slack back` and `lym slack status` set or print the presence, and the Slack texts `away`, `back` and `status` set or print it and are answered in Slack.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D3 bullet 3, Acceptance criteria D3 bullet
    summary: D3 and its criterion have the prime dobby start the relay on each of its passes, but a dobby pass is a batten pass of one run, so with no run in flight no pass happens and a relay that died while away is never restarted
    fix: In D3 bullet 3 and the D3 acceptance criterion, replace "each pass of the prime's dobby" and "a prime dobby pass" with "each tick of the prime's dobby", which runs with or without runs in flight.
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Open risks third bullet
    summary: The Open risks line on thread replies restates D4's Bound that a thread reply is never read and carries nothing new for the plan writer
    fix: Delete the third Open risks bullet, "A thread reply from the phone is never read (D4 bound), so an operator who replies in a thread loses that message."; D4's Bound already states it.
---

### [MEDIUM] The relay is inside its starter's pane tree at its first read

**Location:** `_lym/discussion/decision-record.md` D3 bullet 2, D6 bullet 2, the D3 acceptance criterion.

**Issue:** `proc.Detach` only sets `Setsid` (`internal/proc/proc_linux.go` `Detach`), so the relay's parent stays its starter while the starter lives.
Creel resolves a sender by walking the peer's parent pids (`internal/creelengine/identity.go` `paneOfPeer`), not its session.
`lym slack away` waits up to 30 seconds for the relay's first read, so that read always resolves inside the starter's pane.
From a non-orch strand's pane, `resolveSender` refuses the program claim ("only an orch strand's pane may claim a program name"), and `away` times out.
`lym slack start` races its own exit.
From an orch pane the claim passes only through the orch pane's privilege.
The D3 criterion asserts the false premise, so the implementer is handed a test the design cannot pass.
Three lenses and the handler found this; bound-breaker rated it BLOCKING.
It is kept at MEDIUM: the starters D3 names, the prime bring-up, the prime dobby, the operator's terminal and an orch pane, all succeed, and no PATTERN entry is violated.

### [MEDIUM] A cursorless first start loses messages across a restart

**Location:** `_lym/discussion/decision-record.md` D4 bullets 1-2.

**Issue:** A relay that starts, handles nothing and dies has written no cursor.
Its restart is again a first start from the current time, so a message the operator wrote in between is never forwarded.
That is the dead-relay-while-away case the D3 dobby restart exists for.

### [MEDIUM] A prime anchor change ends the relay

**Location:** `_lym/discussion/decision-record.md` D3.

**Issue:** `internal/fabriccli/anchor.go` calls `deps.EndDobbies(dobbies, runproc.RolePush, runproc.RoleSync)`, which ends every `runproc.HubOwner` process but a push or a sync, and the relay registers under `HubOwner` with a role of its own.
Only dobbies are restarted afterwards.
With presence `terminal` the relay stays down until someone runs `lym slack start`, and the record is silent about it.

### [MEDIUM] `lym slack stop` is undone while presence is `slack`

**Location:** `_lym/discussion/decision-record.md` D3 bullets 3-4.

**Issue:** The prime dobby starts a relay that is down while presence is `slack`, so `stop` lasts one dobby tick.
The record states no outcome and no bound, and the criteria test the two behaviors apart.

### [MEDIUM] The seven-day retention bound is false

**Location:** `_lym/discussion/decision-record.md` D5 Bound line.

**Issue:** `internal/creelengine/journal.go` `retain` carries every unread message, and a program's read, into the new day file before it deletes the old ones (`internal/creelengine/doc.go`, the Storage section).
Mail to `SLACK` therefore waits with no bound, and a relay back after weeks floods the phone with stale mail.

### [MEDIUM] Release-and-repost never drains a persistent failure

**Location:** `_lym/discussion/decision-record.md` D5 bullet 4, D4 bullet 3.

**Issue:** A message Slack keeps refusing at position k reposts messages 1..k−1 on every cycle and never acknowledges.
Each repost resets D4's wait to 5 seconds, so the channel fills with duplicates.
A backlog drained faster than Slack's per-channel posting rate fails in the same way.

### [MEDIUM] The reminder's wait start is ambiguous and survives a resume

**Location:** `_lym/discussion/decision-record.md` D8 bullet 2 and Bound line.

**Issue:** Shed appends a history entry only when a call ends or a goto writes one.
A pause or resume rewrites the status with its history unchanged (`internal/shedengine/run.go` `persistTransient`), and a folded entry keeps its first `At` (`appendOrFold`).
"The entry that made the row current" is the previous row's outcome entry for a routed step and a goto entry for a goto.
A run resumed at PR-Review after a night's pause is reminded at once, against D8's own bound.

### [MEDIUM] The presence verbs have no stated bound

**Location:** `_lym/discussion/decision-record.md` D6 bullets 2-3, D9.

**Issue:** `away` and `back` decide whether the orchs mirror to Slack and whether reminders run.
Nothing in D6 or D9 says who may run them, so any agent strand can run `lym slack back` while the operator is on the phone and cut the operator off.
The rubric's attack-surface rule requires the introducing Decision to state what a new verb lets through and what bounds it.

### [LOW] The rate-limit claim holds only for four hubs

**Location:** `_lym/discussion/decision-record.md` D4 Rationale, Constraints.

**Issue:** At 12 requests a minute per hub, the shared 50-a-minute history limit fits four hubs, and the record states no hub count.

### [LOW] A hold's delay is not bounded by an hour

**Location:** `_lym/discussion/decision-record.md` D4 Bound line.

**Issue:** The same owner renews a hold by holding again (`internal/creelengine/doc.go`, the Holds section).

### [LOW] `@<slug>` mail to a torn-down beam is lost silently

**Location:** `_lym/discussion/decision-record.md` D4 bullet 7.

**Issue:** Program mail to a hub orch with no listed strand stays under the orch's name and is never swept (`internal/creelengine/doc.go`, Receivers and Removed holders), and the operator is never told.

### [LOW] A beam dobby's reminder carries no answer hint

**Location:** `_lym/discussion/decision-record.md` D5 bullet 2, D8 bullet 3.

**Issue:** A reminder from `<shortname>:<slug>:DOBBY` gets no `@<slug>` hint, so an answer in Slack goes to the prime orch.

### [LOW] Slack facts behind the cursor and the text handling are unsourced

**Location:** `_lym/discussion/decision-record.md` D4 bullets 1-2 and 5, Constraints.

**Issue:** The cursor rests on `oldest` being exclusive and on newest-first, paged results, and the record does not state either fact or name a source.
D4 bullet 5 says the text stays as the operator wrote it, while Slack sends URLs and mentions in angle-bracket forms, so a URL typed on the phone reaches the orch as `<https://…>`.
The writer's stencil requires such facts to be settled and recorded with their source.

### [LOW] The outbound rationale and the D6 criterion overstate the Decisions

**Location:** `_lym/discussion/decision-record.md` D5 Rationale, D6 bullet 3 and its criterion.

**Issue:** The relay's own Slack answers bypass creel, and the D6 criterion asks the CLI verbs to answer in Slack, which D6 does not decide.

### [LOW] "Each pass of the prime's dobby" names the wrong cadence

**Location:** `_lym/discussion/decision-record.md` D3 bullet 3, the D3 criterion.

**Issue:** `internal/dobbyengine/doc.go` defines a pass as one run's batten pass, triggered per run.
With no run in flight the prime dobby makes no pass, though it still ticks, so the restart D3 promises never happens.

### [LOW] An Open risk restates a D4 bound

**Location:** `_lym/discussion/decision-record.md` Open risks, third bullet.

**Issue:** The line repeats D4's Bound that a thread reply is never read and adds nothing the plan writer acts on.

## Rejected

- PATTERN-keep-warm-cap applied to `lym slack away`'s 30-second wait: that entry governs the long waits an agent blocks on to keep its prompt cache warm, and a 30-second start timeout is a refusal bound, not such a wait.
- The `pr-review-wait-silent` Open risks line as a board-edit Decision: it asks the parent to mark another entry, which the rubric's own-entry bullet does not cover, and it carries a fact the parent needs.
- Support-log content that lives only there: the Socket Mode reasoning is a rejected alternative, and every lens-review fix the log lists is reflected in the record, so nothing the plan writer needs exists only in the log.
- D2 machine-layer keys shared across hubs: the machine layer is `<Board>/.lym/config/<module>.machine.yaml`, per hub (`internal/configengine/doc.go`), so a per-hub channel id fits it.
