---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D10 (holder-gone sentence)
    summary: The removed-holder sweep never looks at a holder of a read message, so a holder that dies after reading a marked message is never found gone, its frozen clock never runs, and the message is never chased or settled unanswered.
    fix: In D10, replace the sentence that relies on the removed-holder sweep with a stated mechanism, namely that the sweep's subject set and its run condition widen to strands holding a read, marked, unanswered message (internal/creelengine/sweep.go sweepSubjects today takes only unread mail and queued injects), and that the sweep's two-miss rule then settles each such message unanswered with reason holder-gone and answers a waiting --wait-reply. Add to the D10 acceptance criterion a row where the holder holds no other mail or inject when it disappears after reading.
  - id: F2
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks, first bullet
    summary: The board entry says the operator's 2026-10-27 wait for a real mid-turn run of creel hook delivery stands, while the task runs; no Decision for creel-agents note answers it, so only the parent can say whether the run proceeds or holds.
    fix: Parent to answer whether the 2026-10-27 wait is met or whether this run should hold; record the answer as a Decision for creel-agents board note or with lym loom decision add, then drop the Open risks line.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 (clock and restart sentences)
    summary: D10 freezes the chase clock under dormancy, pause and another sender's hold but derives chase state from read time and chased records alone, which cannot rebuild a frozen clock after a restart; and the inject TTL rule it cites does not stop under a hold and survives restarts only through aged records.
    fix: In D10, state that the consumed chase time is journaled the way an inject's consumed TTL is (an aged-style record carrying consumed seconds, written at the inject's persist interval and on stop) and that the restart derivation reads it; reword the parity claim to say the clock stops while the holder is dormant or paused, as an inject TTL does, and additionally under another sender's hold, which the inject TTL does not. Name the record in D11's journal events.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D8 Rationale and Bound
    summary: The rationale says lym shuttle send bypasses creel's pause rules, but send.go reads the pause record and refuses; the bound names only lost guid addressing, while creel inject now queues under a pause and types after the resume, waits for idle, drops the MessageTail and refuses non-orch strands typing into another pane.
    fix: In D8, correct the rationale to say shuttle send refuses under a pause but bypasses creel's journal, queue and hold; extend the Bound to state what the replacement changes, namely that a line injected under a pause is held with its TTL frozen and typed after the resume rather than refused, that it is typed only at idle within its TTL, that no MessageTail is appended unless the sender writes it, and that only EXTERNAL, programs and orch strands may inject into another strand.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 (reply to a chase) and its acceptance row
    summary: A reply to a chase from anyone but the chased message's holder, possible only after the chase was forwarded, goes to the CREEL program, which never reads and is never reminded, so it sits unread for good; D10 calls it an ordinary reply with no stated receiver or bound.
    fix: In D10, replace "from anyone else it is an ordinary reply to the chase" with a refusal of any reply to a chase that is not by the chased message's holder, naming the way forward (post to the original sender by name), or state that chases cannot be forwarded; change the D10 acceptance row "by another strand only the chase" to match.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 Bound
    summary: D2 widens followable trust to every lym program's mail but states the off-Linux identity gap for orch names only; a process with no strand name off Linux, or one detached from the pane tree on Linux, may claim a program, and verbs such as lym loom decision add post caller-supplied text as a program.
    fix: In D2's Bound, add that a program claim is accepted off Linux from any process whose strand name is unset and on Linux from any process outside every pane, that lym verbs posting caller text as a program (lym loom decision add as LOOM) then reach agents as followable mail, and what bounds it (the 0700 lymtemp root, the operator's user); state whether program trust is narrowed to named programs instead.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 and acceptance criterion D1, D3
    summary: For an agent running in a beam worktree, D1 resolves with NotifyFrom from the beam itself while the burler, loom and webster notifiers call orchcli.NotifyParent, which starts above the beam, so the question and the hold notice reach different orchs, contradicting the beam case of the acceptance criterion.
    fix: In D3, state that a runner whose location is a beam or the prime notifies with NotifyFrom's walk, the same rule D1 uses, so the notifier wirings that call NotifyParent unconditionally change; keep the beam case in the D1, D3 acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 and D5 (orch answers a hold notice)
    summary: A held agent sends a marked --parent question and the orch also gets the hold notice, which D3 and D5 tell it to answer with a plain post to the strand; only a --reply by the holder counts as a reply, so creel chases the orch twice and tells the agent its question went unanswered.
    fix: In D5's orch bullet and D3's notice text, say that when the held agent's marked question is in the orch's mail, the orch answers it with lym creel post --reply <id>, and posts to the strand only when no such question is pending; add an acceptance row that a hold-notice answer to a marked question clears the mark.
  - id: F9
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D10 and D12 (journal retention)
    summary: Retention carries only unread messages, queued injects and live holds and prunes the rest after seven days, so a read, marked message whose clock stays frozen that long loses its chase state with no unanswered record, and D12's kept window is left unquantified.
    fix: In D10, state that retention carries a read, marked, unanswered message forward like an unread one, or settles it unanswered with a stated reason at retention; in D12, name the kept window as the journal's seven-day retention (journalRetention in internal/creelengine/journal.go).
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 (restart guarantee, at most two chases)
    summary: The record names no write order between a chase's posted record and its chased record, so a crash between the two lets a restarted daemon post a third chase, breaking both the no-repeat guarantee and the two-chase bound.
    fix: In D10, state that the chase number travels on the chase's own posted record, or that the chased record is journaled before the chase is posted and a restart counts it, so a crash never yields a third chase.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 Bound (chase carries no sender text)
    summary: The chase body names the original sender, and an EXTERNAL sender renders with its client-claimed envelope (terminal, parent process, directory), so a directory name can carry text into a CREEL-signed chase that D2 and D6 tell agents to follow.
    fix: In D10, state that a chase and the sender notice name an EXTERNAL sender as the bare EXTERNAL name without its envelope.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 Bound (never-read marked message)
    summary: The bound says a never-read marked message is covered by the reminder backoff and the removed-holder sweep, but the reminder backoff applies to pane-mail strands only; a live hook strand gets one ring and at most one bell per set, so its never-read marked message is never chased and a waiting sender waits indefinitely.
    fix: In D10's Bound, state that a never-read marked message held by a live hook strand gets one ring and one bell per claimable set and nothing more, and that a waiting --wait-reply sender is bounded only by its own call timeout.
  - id: F13
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D12
    summary: D12 reads the report through the Go client's log call, but a log entry carries no reply-to link, so the report cannot tell replies to marked messages from other replies.
    fix: In D11 or D12, state that the log entry carries the reply-to id of a reply alongside the mark, so lym report replies counts a reply against the marked message it answers.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 Rationale
    summary: D5's rationale says every question carries the mark D8 chases, but D8 removes the shuttle verbs and D10 is the chase decision.
    fix: In D5's Rationale, change "the mark D8 chases" to "the mark D10 chases".
  - id: F15
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md Constraints, creelcli and orchcli cycle line
    summary: The cycle line says D1's walk and orch liveness check must be reachable from creelcli without the cycle but records no evidence that they can be; research shows creelcli already depends on orchengine, hubgeom, reedengine and fabricengine, which OrchLive and OrchStrandOf need.
    fix: Extend the constraint with the evidence that go list -deps ./internal/creelcli already includes internal/orchengine, internal/hubgeom, internal/reedengine and internal/fabricengine, so the walk and orchcli's OrchLive and OrchStrandOf can move below orchcli without a new import edge.
  - id: F16
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Scope, parent directive bullet
    summary: The Scope bullet carries deliberation narrative about the operator's 2026-10-23 view and the extra read turn, which belongs in the support log; the requirement that Claude-to-Claude messages move too must stay.
    fix: Cut the bullet's second sentence to "This covers Claude-to-Claude messages too, so one route serves every engine." and move the reasoning about hook delivery removing the extra read turn into support-log.md's Rejected alternatives entry on each engine's own messaging.
---

### [BLOCKING] Holder gone after reading is never detected

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** D10 says a holder the removed-holder sweep finds gone makes the message `unanswered` at once.
The sweep's subjects are only strands holding unread mail or a queued inject (`internal/creelengine/sweep.go`, `sweepSubjects` checks `msg.Status == StatusUnread`), and it runs only while such a subject exists.
A holder that read a marked message and holds nothing else is never swept; D10's clock counts only while the holder is live, so it freezes, no chase is posted, and the message stays owed forever, with a `--wait-reply` sender waiting without end.
Found by the existing-paths and walkthrough lenses.

### [BLOCKING] Parent question: is the 2026-10-27 wait met?

**Location:** `_lym/discussion/decision-record.md` Open risks

**Issue:** The board entry states twice that the operator's 2026-10-27 wait, a real mid-turn run of creel's hook delivery, stands, and that the `creel-hook-post` real-Claude scenarios did not run.
The task is nevertheless running, and `lym board list` shows no `Decision for creel-agents:` note answering it.
This is a real parent question, not one the writer could settle; the writer was right to raise it rather than assume it.

### [MEDIUM] Chase clock cannot be rebuilt after a restart

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** The inject TTL survives a restart only through `aged` records with `ConsumedSeconds`, persisted every `agedPersistInterval` (`internal/creelengine/deliver.go`, `ageInjects`, `persistAged`), and `ageInjects` stops only for a dormant or paused receiver, never for a hold.
D10 derives chase state from read time and `chased` records alone, so after a pause and a restart the daemon counts paused time and chases early; its claim to reuse the inject TTL's rule is wrong on the hold.
Found by three lenses.

### [MEDIUM] D8 misreads shuttle send and under-states its bound

**Location:** `_lym/discussion/decision-record.md` D8

**Issue:** `internal/shuttlecli/send.go` reads the pause record and refuses with the pause's way forward, which `PATTERN-pane-text-injection`'s background also records.
`lym creel inject` instead holds the line under a pause and types it after the resume, waits for idle, appends no `MessageTail`, and lets only EXTERNAL, programs and orch strands inject into another strand.
The Bound names only the lost guid addressing, so the rubric's attack-surface rule is not met.

### [MEDIUM] Reply to a forwarded chase lands in a mailbox nobody reads

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** `Store.Reply` (`internal/creelengine/verbs.go`) checks only that the replier holds the message, then appends to `original.From` with no receiver check.
A chase's sender is `<shortname>:CREEL`, which never reads and whose mailbox is never reminded, so a reply by a forwardee of the chase is stored and never seen.
This is a new routing edge with no stated bound.

### [MEDIUM] Program trust gap stated only for orch names

**Location:** `_lym/discussion/decision-record.md` D2

**Issue:** `resolveFromEnvironment` (`internal/creelengine/identity.go`) accepts a program claim off Linux whenever the strand name is unset, and on Linux a process in no known pane may claim a program.
`lym loom decision add` posts caller text as `LOOM`, so after D2 and D6 make every program's mail followable, an off-Linux Codex agent can pose as a program, not just as an orch.
The Bound names only the orch case.

### [MEDIUM] Beam agents' question and hold notice diverge

**Location:** `_lym/discussion/decision-record.md` D3; acceptance criterion D1, D3

**Issue:** D1 resolves from a beam with `NotifyFrom`, starting at the beam, while the burler, loom and webster notifiers call `orchcli.NotifyParent(location)`, whose `parentOrchStrand` starts at the location's recorded parent.
For a held agent in a beam the two reach different orchs, which contradicts D3's claim and the beam case of its acceptance criterion.

### [MEDIUM] A hold-notice answer by plain post leaves the marked question owed

**Location:** `_lym/discussion/decision-record.md` D3, D5

**Issue:** The held agent posts its question with `--parent --expect-reply`, and the orch also gets the hold notice telling it to post to the strand.
D9 counts only a `--reply` by the holder, so an orch that follows the hold notice is chased twice, and the agent gets an `unanswered` notice that D2 and D6 tell it to act on.

### [LOW] Retention drops owed read messages

**Location:** `_lym/discussion/decision-record.md` D10, D12

**Issue:** `retain` (`internal/creelengine/journal.go`) carries only unread messages, queued injects and live holds past `journalRetention`, seven days, and prunes the rest.
A read, marked message whose clock is frozen that long is dropped with no outcome; D12's "kept window" is that same retention and is left unnamed.

### [LOW] Crash between chase post and chased record

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** With no stated order between a chase's `posted` and `chased` records, a crash between them lets a restarted daemon post a third chase.

### [LOW] EXTERNAL envelope can reach a chase body

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** An EXTERNAL sender renders with its client-claimed envelope (`internal/creelengine/seam.go`), so naming the sender in the chase can carry client-chosen text under the `CREEL` signature.

### [LOW] Never-read bound is wrong for hook strands

**Location:** `_lym/discussion/decision-record.md` D10

**Issue:** The reminder backoff is the pane path's; a hook strand gets one ring and at most one bell per claimable set, so the stated cover does not hold for it.

### [LOW] Log entries carry no reply link

**Location:** `_lym/discussion/decision-record.md` D12

**Issue:** `LogEntry` (`internal/creelengine/fold.go`) has no reply-to field though the journal `record` has `ReplyTo`, so the report verb cannot pair a reply with the marked message it answers.

### [LOW] D5 cites the wrong decision for the chase

**Location:** `_lym/discussion/decision-record.md` D5

**Issue:** "the mark D8 chases" points at the shuttle verb removal; the chase is D10.
Every other D-number reference in the record, acceptance criteria included, points at the decision that states the thing.

### [LOW] Cycle constraint records no evidence

**Location:** `_lym/discussion/decision-record.md` Constraints

**Issue:** The cycle itself is real: `internal/orchcli/notify.go` imports `internal/creelcli`.
The line leaves open whether the walk can be reached without it, but `go list -deps ./internal/creelcli` already lists `internal/orchengine`, and `creelcli` imports `hubgeom`, `reedengine` and `fabricengine` directly, which `OrchLive` and `OrchStrandOf` need; the record should carry that answer.

### [NIT] Deliberation in Scope

**Location:** `_lym/discussion/decision-record.md` Scope

**Issue:** The clause about the operator's 2026-10-23 view and the extra read turn is reasoning, not a requirement; the plan writer needs only that Claude-to-Claude messages move too.
The completeness-before-leanness test keeps that requirement in the record.

## Rejected

- Walkthrough's severity of LOW for the reply-to-chase defect: merged into F5 at MEDIUM, since the rubric's attack-surface rule makes an unbounded routing edge more than a nit.
- Existing-paths' severity of MEDIUM for the retention edge: merged into F9 at LOW, since it needs a holder frozen for seven days.
