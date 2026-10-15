---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D9 and D6 (inject delivery across a daemon crash or binary-replacement exit)
    summary: An inject in flight when the daemon dies is rebuilt as still queued and retyped on restart, the double-run D9 forbids, and no Decision or criterion covers it
    fix: In D9, add that the daemon appends and syncs a sending event for an inject before it calls shuttle's send, and that the fold treats an inject with a sending event and no typed, unconfirmed or expired outcome as unconfirmed and never retypes it. Add sending to D6's event list. In D2, say whether the binary-identity exit waits for in-flight sends to finish before it closes the listener. Under Acceptance criteria, add that a daemon killed after an inject's send began and before its outcome was journaled journals the inject as unconfirmed on restart and never types it again.
  - id: F2
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md D4 (program sender vs. ancestry precedence)
    summary: D4 leaves open whether a client-API program-sender name or the Linux ancestry wins for a peer inside a strand pane, so a process in an agent pane can either claim a privileged upper-case name or batten run from the orch pane signs as the orch
    fix: In D4, decide the precedence and state it as a rule. For example, on Linux a peer inside a strand pane is always that strand, so a program-sender name claimed from inside a pane is refused (or ignored), and a program sender is accepted only from a peer in no strand pane. Then add to D4's Bound which program-sender names are accepted (any upper-case name, or a closed set declared in `creelengine`) and that any process of the OS user outside a pane can claim one and gain D9's privilege. Under Acceptance criteria, add that on Linux a program-sender claim from inside a strand pane resolves by the chosen rule.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 (Bound and Rationale)
    summary: On Linux a peer in no strand pane is trusted on its LYX_STRAND_NAME, so a one-command detach such as setsid -f with the env unset or set to an orch name escapes the ancestry walk, yet the Rationale says ancestry closes the bypass and the Bound names only non-Linux platforms
    fix: In D4, add that on Linux a peer in no strand pane whose LYX_STRAND_NAME names a strand of this hub is refused, since every live strand's pane process is known. Extend D4's Bound to say that on Linux a process detached from its pane's ancestry (setsid, a double fork, tmux run-shell) passes as EXTERNAL and gains D9's privilege. Reword the Rationale so it claims to close only the in-pane env drop. Widen the matching Open risks line from off Linux to every platform for detached processes. Under Acceptance criteria, add that on Linux a peer outside every strand pane claiming a strand name is refused.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9 and D2 (inject TTL across a restart)
    summary: The TTL counts only live, unpaused time, but no journal event records the consumed TTL, so D2's rebuild with their remaining TTL cannot be derived from the journal and whether daemon downtime counts is unstated
    fix: In D9, state how the consumed TTL is persisted, for example an expires-after value the daemon rewrites as a new journal event whenever the receiver goes dormant, paused or live and when the daemon stops, or a remaining-TTL field on the carried record. Also say whether time with the daemon down counts toward the TTL. Add any new event to D6's event list. Under Acceptance criteria, add that a daemon restarted mid-TTL types or expires a queued inject at the same consumed TTL it held before the restart.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D12 (reconnecting waiters)
    summary: An outcome that lands while a waiter is disconnected is lost to it, since a reply with no waiter is journaled undelivered and D12 says the reconnecting waiter only waits on, never reads back, the same id
    fix: In D12, add that a reconnecting waiter is answered at once from the pool when the outcome it waits on already happened while it was disconnected. That covers an undelivered reply to its message id, which is handed over and marked delivered, and an inject's typed, unconfirmed or expired outcome. Under Acceptance criteria, extend the each-waiting-verb-reconnects line so that a reply or an inject outcome journaled while the waiter was disconnected reaches it on reconnect.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D15 and D6 (queued injects of a removed holder)
    summary: D15 moves only unread mail off a gone holder, while a queued inject's TTL never runs for a non-live receiver, so injects to a torn-down strand are carried forward by D6 forever and an inject --wait on one never returns
    fix: In D15, add that when a holder is gone, each inject queued for it is journaled as expired (or a new undeliverable outcome) and never typed, which answers any inject --wait on it. Extend the removed-holder Acceptance criterion to cover a queued inject to a removed strand ending with that outcome and leaving the carry-forward set.
  - id: F7
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10 and D13 (interrupt under a hold)
    summary: D13 mutes other senders' injects and reminders but says nothing of interrupt, so a non-holder can interrupt a held receiver mid-cycle, and inject --interrupt from a non-holder interrupts and then has its text muted until the hold ends
    fix: In D10, state that interrupt and inject --interrupt from a sender other than the receiver's holder are refused while a hold covers the receiver, naming the hold's holder and expiry. Otherwise, if interrupts are meant to pass a hold, say so and bound it in D13. Under Acceptance criteria, add the chosen behavior for an interrupt from a non-holder under a hold.
  - id: F8
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D13 (several holders on one receiver)
    summary: D13 says one sender's hold on one receiver but leaves open whether a second sender may hold the same receiver and, if so, whose injects and reminders pass
    fix: In D13, decide one rule. Either a hold on an already-held receiver by another sender is refused, naming the current holder and expiry, or several holds coexist and only senders that hold pass. State it with its bound. Under Acceptance criteria, add the case of a second sender's hold on a held receiver.
  - id: F9
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Several observable Decision behaviors have no criterion, so a plan writer must guess whether to cover them, among them D18's log filters and its never-bodies rule
    fix: "Under Acceptance criteria, add these lines: lyx creel log filters by --id, --from, --to and --since and never prints a body (D18); a reminder lists at most 5 messages and counts the rest, and a subject is cut to 80 runes with control and delimiter characters replaced (D7); the reminder backoff doubles from 5 minutes to a 1-hour ceiling (D7); on Linux a LYX_STRAND_NAME that disagrees with the ancestry is refused (D4); every creel verb refuses in standalone geometry, naming that creel needs a hub (D2); a receiver whose sends stay busy delays no other receiver's delivery (D19)."
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Question ledger Q8
    summary: Q8 records the inject TTL as wall time, while D9 counts it only while the receiver is live and unpaused and Rejected alternatives rejects a wall-time TTL; the ledger entry is stale
    fix: In support-log.md's Question ledger, change Q8's pick to 10 minutes by default and 1 hour at most, counted only while the receiver is live and unpaused (D9, D14).
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D9 vs D16 and the Acceptance criteria
    summary: D9 and its criterion say any sender may inject into its own strand, while D16 and its criterion refuse every inject into an orch, so an orch's self-inject meets two contradictory rules
    fix: In D9, qualify the self-inject sentence so it reads that any sender but an orch may inject into its own strand (D16). In the Acceptance criterion that says a self-inject passes, add except an orch's.
  - id: F12
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D9 and D16 (role orch)
    summary: D9 and D16 key privilege and the orch exclusion on role orch, but agentname numbers repeated roles as orch-2 and fabric's role guard matches them with MatchesRole, so whether a numbered orch is privileged and excluded is left open
    fix: In D9 and D16, say that a strand whose role matches orch under agentname.MatchesRole (orch or orch-N) is an orch strand for both rules, or state the opposite choice explicitly.
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Notes for the plan writer, third bullet
    summary: The note says no code under internal/ listens on or dials a socket, but internal/reedengine/socketfile.go dials a unix socket to remove a stale socket file, the very step D3 describes
    fix: In Notes for the plan writer, replace the third bullet with one saying no production code under internal/ listens on a socket, and that internal/reedengine/socketfile.go's stale-socket removal (dial, and on no answer re-stat and remove) is the model for D3's removal of an unanswered socket file.
  - id: F14
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md D3 Rationale, last sentence
    summary: The clause that a cross-hub feature finds the socket by shortname and hub-path check designs for cross-hub sending, which Scope puts out, and adds nothing the plan writer builds
    fix: In D3's Rationale, end the last sentence after no longer holds, dropping the cross-hub clause. The deliberation behind it stays in support-log.md's Turn 3 leanness entry.
---

### [BLOCKING] An in-flight inject is retyped after a crash

**Location:** `_lyx/discussion/decision-record.md` D9, D6, D2

**Issue:** D9 journals an inject as injected or unconfirmed only after shuttle's send returns, and D2 rebuilds queued injects on start.
A daemon killed, or exiting on a replaced binary, after the keys went out and before the outcome line was synced leaves the inject queued in the fold, so the restarted daemon types it again.
That is the double run of a command or `/clear` that D9's "never retyped" rule and Rejected alternatives exist to prevent.
D6's event list has no event marking a send as begun, and the restart criterion ("rebuilds the pool … queued injects included") would pass on the faulty behavior.

### [BLOCKING] Program-sender claim vs. ancestry precedence is undecided

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** D4 says a peer inside a strand pane "is that strand", and also that a Go program names itself through the client API as `<shortname>:<NAME>`.
It never says which wins when both apply.
`lyx batten run` is started from the orch's pane, so its process descends from the orch pane.
If ancestry wins, `ly:BATTEN` signs as the orch and program senders never exist on Linux.
If the claim wins, any process inside an agent pane (a small `go run` against the client API, which agents in this repo can write) claims an upper-case name and gains D9's privilege, the escalation D4 says it closes.
No bound states which upper-case names are accepted.
Since privilege rides on it, the plan writer must not guess this.

### [MEDIUM] The Linux ancestry walk is escapable by detaching

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** For "a peer in no strand pane", D4 falls back to `$LYX_STRAND_NAME` on Linux too.
A process re-parented away from its pane (`setsid -f`, a double fork, a `nohup` child outliving the Bash call, `tmux run-shell`) is in no pane's ancestry.
So `setsid -f env -u LYX_STRAND_NAME lyx creel inject …` becomes EXTERNAL, and `setsid -f env LYX_STRAND_NAME=ly:orch …` becomes an orch.
Both are privileged, and each is one command.
The Rationale's claim that ancestry "closes the one-command bypass" overstates it, and the Bound covers non-Linux platforms only.
Impersonating a named strand from outside its pane can be refused outright on Linux, since D1's seam lists every strand's pane process.

### [MEDIUM] The inject TTL cannot survive a restart as decided

**Location:** `_lyx/discussion/decision-record.md` D9, D2, D6

**Issue:** D9 counts the TTL only while the receiver is live and unpaused, and D2 promises queued injects come back "with their remaining TTL".
None of D6's events records liveness or pause transitions or consumed TTL, so the fold can derive neither.
A plan writer must choose between resetting the TTL on restart (a stale `/clear` lands late) and counting wall time (the rejected alternative).
Whether daemon downtime counts is also unstated.

### [MEDIUM] A reconnecting waiter misses an outcome from its disconnected window

**Location:** `_lyx/discussion/decision-record.md` D12

**Issue:** After a daemon restart, the reply's `post` can start the daemon and land before the `--wait-reply` client reconnects.
D12 then journals the reply as undelivered, and the reconnecting waiter "waits on the same id" forever.
The same gap holds for an `inject --wait` whose inject was typed or expired while it was disconnected.
The restart criterion ("each waiting verb reconnects and keeps waiting") pins the defect rather than catching it.

### [MEDIUM] Queued injects of a removed holder never resolve

**Location:** `_lyx/discussion/decision-record.md` D15, D6, D9

**Issue:** D15's sweep runs "while unread mail exists" and moves only unread messages.
An inject queued for a strand whose pair was torn down never expires, since D9's TTL runs only while the receiver is live.
D6 carries it into each new day file for good, and an `inject --wait` on it blocks until the client's reconnect cap.
This is restart-surviving state that never resolves.

### [MEDIUM] Interrupt passes a hold

**Location:** `_lyx/discussion/decision-record.md` D10, D13

**Issue:** D13's hold exists for the watcher's context cycle and mutes other senders' injects and reminders.
D10's interrupt is "at once, bypassing the queue", is refused only under a pause, and never mentions holds.
A privileged non-holder can therefore stop a held receiver's turn mid-cycle.
`inject --interrupt` from a non-holder interrupts and then has its head-of-queue text muted by the hold, leaving the receiver stopped with no instruction.
Under the attack-surface rule, D10 must state what it skips here and what bounds it.

### [MEDIUM] Several holders on one receiver

**Location:** `_lyx/discussion/decision-record.md` D13

**Issue:** "One sender's hold on one receiver" does not say whether a second privileged sender may hold the same receiver.
If two holds coexist, each holder's injects come from "a sender other than the holder" of the other hold, and the record does not say whether they pass.
The plan writer has to invent the rule.

### [MEDIUM] Decision behaviors with no Acceptance criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** D18 has no criterion at all, though its never-bodies rule is what keeps "only the holder reads" true.
D7's subject sanitizing is the prompt-injection mitigation that Open risks leans on, and its 5-message cap and 1-hour backoff ceiling have no criterion either.
Other untested behaviors: D4's refusal of a disagreeing `LYX_STRAND_NAME` (only the unset case is covered), D2's standalone refusal, and D19's per-receiver independence.
D8's ack and D11's forward are covered.

### [LOW] Question ledger Q8 is stale

**Location:** `_lyx/discussion/support-log.md` Question ledger Q8

**Issue:** Q8 reads "wall time", but Turn 3's goal-scope entry changed the TTL to count only live, unpaused time, and Rejected alternatives rejects a wall-time TTL.
The decision record means live, unpaused time (D9, its Auto-mode line, and the criterion "its TTL does not run while the receiver is paused or dormant").
The ledger entry was not updated.

### [LOW] Orch self-inject meets two rules

**Location:** `_lyx/discussion/decision-record.md` D9, D16, Acceptance criteria

**Issue:** D9 says "Any sender may inject into its own strand", and D16 says inject into an orch is refused.
Their criteria repeat both.
I found no path where one orch's inject reaches another orch: D16 refuses every inject and interrupt into any orch strand, and holds and mail type nothing.
Only an orch's self-inject is left ambiguous.

### [LOW] Numbered orch roles

**Location:** `_lyx/discussion/decision-record.md` D9, D16

**Issue:** `agentname.NumberRole` forms `orch-2` for a repeated role, and `internal/fabriccli/roleguard.go` treats such a role as an orch through `agentname.MatchesRole`.
D9 and D16 say "role `orch`" without saying whether a numbered form is included.

### [LOW] False plan-writer note on sockets

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer

**Issue:** `internal/reedengine/socketfile.go` calls `net.Dial("unix", path)` and removes a socket file that no listener answers.
That is D3's stale-socket step, so the note "D3's transport has nothing to copy" hides a model to copy.
The Open risks claim that creel is the first socket server holds, since nothing in production listens.

### [LOW] Out-of-scope cross-hub clause in D3

**Location:** `_lyx/discussion/decision-record.md` D3 Rationale

**Issue:** Scope puts cross-hub sending out.
"A cross-hub feature finds it by shortname and hub-path check" is design for that excluded feature.
Neither a requirement nor a constraint the plan writer needs depends on it, so cutting it loses nothing.
The supersession of the board premise stays.

## Rejected

- D3 and D7's board-departure sentences as relocation candidates: each tells the plan writer not to follow the board on a named point, which is a requirement, so they stay in the record. Only D3's cross-hub clause is flagged (F14).
- Auto-mode assumptions repeating Decisions: the spec makes the section required, and its lines are one-line pointers at the Decisions, so this is not padding.
- D17's widening of `Runner.Send` and `Runner.Interrupt`: its Bound states the reach (any live reed strand in the worktree, behind the pane guard) and the held-box consequence. The orch strand is a shuttle run with a record, which `orchcli/session.go`'s use of `Runner.Send` shows, so the widening opens no new path into an orch.
- D8's at-least-once reread after a crash between print and ack: a duplicate read is benign and is the deliberate trade against losing mail.
