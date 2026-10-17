---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 Restart bullet
    summary: The restart journal lookup spans the whole phase, so a restarted watcher in resuming adopts an earlier step's inject and skips the pending step, the resume pointer included
    fix: In D5's Restart bullet, replace "since the phase began" with "since the pending step was persisted", persist that time with the pending step, and adopt only an inject whose command name and argument (or text) match the pending step; state that the skills and retry steps, which can carry the same load-skills argument, are told apart by that persist time. Extend the restart acceptance criterion with a restart in the resuming phase at each reload step, the pointer and a retry of every skill included.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5 Hold and Restart bullets
    summary: A fresh launch or adopt resets the phase in the CLI, so the dead cycle's hold and its queued /clear, /compact or pointer injects on the fixed orch name survive and creel types them into the relaunched session
    fix: Add to D5 that `lyx orch start` (relaunch, --continue and --adopt), wherever it calls ResetForFreshLaunch, releases ORCHWATCH's hold on the orch strand and expires every inject ORCHWATCH has queued for it before the new session is typed into, and say which creel call does that (a new withdraw verb for a program's own queued injects, stated with its bound, or the release plus a requirement that a command inject is bound to the strand GUID it was queued for and expires on a GUID change). Add an acceptance criterion - an orch relaunched mid-cycle gets none of the old cycle's injects and its mail is reminded at once.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5 Hold bound and advance-rule bullets
    summary: The color and plugins steps have no timeout and no unconfirmed rule, so the stated hold bound (phase timeouts everywhere but clearing) and every other unconfirmed step is gated by its own signal are false and an unconfirmed or repeatedly expiring plugins step holds the orch's mail without end
    fix: In D5's advance rule, state that unconfirmed advances the color and plugins steps and the bind color too, as today's no-confirmation steps do; give the color and plugins steps the handoff timeout from their first queueing, after which the reload moves on as a skills timeout does; correct the Hold bound to name the bound that now covers every phase; add an acceptance criterion for an unconfirmed and for a repeatedly expired plugins inject.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 reply handling and D5 Restart bullet
    summary: The watcher's per-tick read acknowledges the written reply before the gate's result is persisted, so a watcher that dies in between loses the reply and the restarted watcher aborts a cycle whose note is written
    fix: Add to D4 or D5 that the watcher persists the gate's outcome (the phase move, or the deferral) before it acknowledges the read, or that a restarted watcher in handoff-requested holding the request id finds a reply to that id in creel's journal and gates on it; add the case to the restart acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 transcript bullet
    summary: Steps that wait on their own turn end now queue an inject that creel types later, and D5 never says the step's events offset is taken at typing, so a turn end between queueing and typing confirms the skills, retry or pointer step
    fix: Add to D5 that a step waiting on its own signal reads only events after creel's sending record for its inject (the watcher takes the events offset when it reads the inject's outcome, or reads the sending time from the journal), never from the queueing time, and add an acceptance criterion with an operator turn between queueing and typing.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 Hold bullet and auto-compaction bullet
    summary: The hold is taken at a cycle start only, so the auto-compaction reload enters resuming with no hold and creel may type reminders and foreign injects between its reload steps
    fix: In D5's Hold bullet, say the watcher takes the hold on entering any non-idle phase, the auto-compaction reload's resuming phase included, and add that case to the hold acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D6 Sender paragraph
    summary: Detached NotifyPrime callers (hibernate watch, a detached loom driver) inherit the spawning pane's strand name but run in no pane, and creel refuses a strand claim from outside a pane, so the hibernate notice the acceptance criteria require is refused
    fix: In D6, state that NotifyPrime signs as the caller's strand only when the calling process runs in that strand's pane, and otherwise sends a program claim with no strand claim, as D10 does for the watcher; add a detached hibernate watch spawned from the orch pane to the sender acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 bound and D5 creel-failure paragraph
    summary: Creel refuses every out-of-pane sender while any live strand's pane PID is unknown or any worktree's reed state is unreadable, so a hub-wide strand fault blocks the watcher's hold and every hard cycle, contradicting D1's only-lyx-code-can-block claim with no stated bound
    fix: In D1's bound, drop the claim that only lyx code can block the orch's cycles and name this refusal; in D5, state the bound for it, either a hard cycle that proceeds without the hold when creel refuses the hold for a sender-resolution reason, or an acceptance that the cycle waits and lyx orch status shows the refusal as State.Stuck, and add the case to the creel-failure acceptance criterion.
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 wake-up sentence against D2
    summary: D6 has the role read with lyx creel read --all while D2's held reminder names read --from the owner, so an orch woken mid-cycle reads the held batten notices and works them before the note
    fix: In D6, say the role reads with the command the reminder names, which is read --from the owner under a hold and read --all otherwise, and that the stencil says so.
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 bound and D8 fallback
    summary: The D6 bound says unread mail to a removed orch returns to its program sender and is journaled undelivered, but creel returns a strand-signed notice into the sending strand's mailbox and reminds it there, and D8's fallback reads only BATTEN's log
    fix: Restate D6's bound for both signers - a program-signed notice is journaled undelivered, a strand-signed one returns to that strand's mailbox - and either have D8's fallback read lyx creel log --to the orch strand instead of --from BATTEN, or state that the landed notice is always program-signed and why.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4 answer forms
    summary: The answer form lyx creel post --reply id written path passes two arguments, and post --reply refuses anything but one text argument
    fix: In D4, write the answers quoted, lyx creel post --reply <id> "written <path>", and keep defer as one word.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D9 pause-resume line
    summary: The pause-resume line becomes a TTL inject that expires if the orch stays busy past the TTL, the entry is already marked noticed, and D5's expired requeue covers cycle steps only
    fix: In D9, state the resume line's TTL (the one-hour maximum) and that the watcher tracks its inject id and queues it again on expired, or that it goes as mail instead.
  - id: F13
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7
    summary: D7 never says which sender tender's revive inject carries, and a strand sender from a non-orch pane is refused by creel's checkMayAct
    fix: In D7, state that tender injects as its program, TENDER, with no strand claim, and that a revive run from a non-orch strand's pane is refused with the way forward.
  - id: F14
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 bound
    summary: Two concurrent lyx tender revive calls both find no standing revive and both inject, so the injects-none-while-one-stands claim fails under concurrency
    fix: Add to D7's bound that two concurrent revives can each inject, costing at most one extra turn within the TTL, or serialise the check and inject under a tender lock.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md Scope In, observable CLI changes
    summary: lyx tender revive's report keeps its conditional and watcher-live claims, which D7's creel path makes false, and Scope's observable CLI list omits the change
    fix: Add to Scope In and D7 that the revive report drops the conditional-delivery and watcher-live fields for the orch and reports the inject id, and that tenderengine's doc says so.
  - id: F16
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D6 and D4
    summary: The orch state records the strand's GUID, while creel addresses and signs by strand name, and neither D6's posting nor D4's sender check says how the GUID becomes the name
    fix: In D6 and D4, state that the orch strand's name is resolved from its GUID through reed's state at each call, and that D4 compares the reply's sender with that resolved name.
  - id: F17
    severity: LOW
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md Scope In, observable CLI changes
    summary: lyx orch status has no hold or send stuck fields; StuckByHold and StuckBySend are state-file keys only, so the stated CLI change rests on a false premise and the real change, the stuck help text, is missed
    fix: Replace the clause with - the orch state loses its stuck_by_hold and stuck_by_send keys, and lyx orch status's help text says stuck names the stall reason creel last reported for the watcher's pending inject.
  - id: F18
    severity: LOW
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D2 stencil sentence
    summary: D2 declines a stencil line because a reminder never interrupts a turn, but the board's open point already grants that and asks about the next turn a typed reminder takes
    fix: Replace the reason with one that answers the open point - when an agent reads its mail is part of stage 3's reply-discipline rules, which Scope Out leaves to creel-agents - and update Q3 in the support log to match.
---

### [BLOCKING] Restart lookup adopts an earlier step's inject

**Location:** `_lyx/discussion/decision-record.md` D5, Restart bullet.

**Issue:** The lookup reads ORCHWATCH's posts and injects to the orch "since the phase began". The resuming phase is entered once in `startReload` and `advanceReload` never moves its start (`internal/orchengine/watcher.go:1027-1035`, `1135-1146`), so the color, plugins, skills and retry injects all fall in the window. A watcher that dies with the pointer step pending and no id adopts an earlier `injected` inject, never types the pointer, and leaves with "resume timed out" in a cleared session. When every skill was missing, `settleSkillTurn` (`watcher.go:1170`) gives the retry step the full skill list, so even matching on command and argument confuses the skills and retry injects.

### [BLOCKING] A fresh launch leaves the dead cycle's hold and queued injects on the orch name

**Location:** `_lyx/discussion/decision-record.md` D5, Hold and Restart bullets.

**Issue:** The orch strand keeps a fixed name across relaunches (`internal/orchcli/start.go:96`), and creel keys holds and queues by name. `lyx orch start`'s relaunch and adopt reset the phase to idle in the CLI through `ResetForFreshLaunch` (`start.go:304`, `start.go:367`, `internal/orchengine/state.go:313`), so D5's re-take and release rules never fire. The old `/clear`, `/compact` or pointer inject, owned by ORCHWATCH and so passing its own leftover hold, is typed into the resumed session within its TTL. Creel has no verb that withdraws a queued inject, and the leftover hold keeps batten's notices unreminded for up to an hour with no cycle running.

### [BLOCKING] The color and plugins steps escape both the unconfirmed rule and the stated bound

**Location:** `_lyx/discussion/decision-record.md` D5, Hold bound and advance-rule bullets.

**Issue:** `tickResuming`'s contract says "The color and plugins steps have no confirmation and no timeout" (`internal/orchengine/watcher.go:1176-1181`). D5 advances on `unconfirmed` only for `/clear` and steps waiting on their own signal, and the Open risks name `/reload-plugins` as possibly `ErrNotLanded` (`internal/shuttleengine/send.go:596`). An unconfirmed plugins inject has no rule, and an expiring one is queued again with no end, while the hold is renewed every tick. The bound "the existing phase timeouts bound everywhere but the clearing phase" is false, so the hold this Decision introduces has no true bound.

### [MEDIUM] A read reply is lost on a restart

**Location:** `_lyx/discussion/decision-record.md` D4 and D5 Restart.

**Issue:** The Go client acknowledges a read inside the call (`internal/creelengine/client.go:269`), and `Ack` journals the message read (`verbs.go:168`). A watcher dying between the read and saving the clearing phase finds an empty mailbox after restart and waits out the handoff timeout. The old watcher re-stats the note file each tick (`watcher.go` `tick`, `handoffWritten`) and never lost a written note.

### [MEDIUM] Step events offset is taken at queueing

**Location:** `_lyx/discussion/decision-record.md` D5, transcript bullet.

**Issue:** `typeReloadStep` sets `PhaseEventsOffset` at typing so that "a turn end read before it never confirms the step". Under D5 the watcher queues and creel types later; an operator turn in between ends a turn the skills, retry or pointer step reads as its own.

### [MEDIUM] The auto-compaction reload runs unheld

**Location:** `_lyx/discussion/decision-record.md` D5, Hold and auto-compaction bullets.

**Issue:** The hold is taken "at a cycle start"; the auto-compaction entry (`startAutoReload`) enters resuming with no cycle start. Today `deliverNotice` types nothing outside `PhaseIdle`; without a hold creel types reminders and foreign injects between the reload steps.

### [MEDIUM] Detached notifiers are refused by creel's identity rule

**Location:** `_lyx/discussion/decision-record.md` D6, Sender.

**Issue:** `hibernate watch` and a detached loom driver keep the spawning pane's environment and are reparented out of the pane tree. The CLI's `claimFor` builds the strand claim from the environment (`internal/creelcli/cli.go:209`), and `resolveSender` refuses a strand claim from a process in no pane (`internal/creelengine/identity.go`). D6's rule signs such a caller as the strand, or as the program while still carrying the inherited strand claim; either way the hibernate notice the acceptance criteria name is refused.

### [MEDIUM] A hub-wide strand fault blocks every cycle

**Location:** `_lyx/discussion/decision-record.md` D1 bound, D5 creel-failure paragraph.

**Issue:** `paneOfPeer` refuses every out-of-pane sender while any live strand's pane PID is unknown or `Strands()` fails on any worktree (`identity.go`, `internal/creelcli/delivery.go:96-113`). ORCHWATCH's hold is then refused every tick and no hard cycle starts. D1 claims only lyx code can block the orch's cycles; D5 files this as an outage and states no bound for it.

### [MEDIUM] The role's read --all undoes the held reminder's read --from

**Location:** `_lyx/discussion/decision-record.md` D6 against D2.

**Issue:** D2 names `read --from <owner>` so the orch reads only the watcher's mail mid-cycle. D6 tells the role to read with `--all`, which returns every unread message (`internal/creelengine/doc.go`, read), the held batten notices included.

### [MEDIUM] Strand-signed notices do not return as D6's bound says

**Location:** `_lyx/discussion/decision-record.md` D6 bound, D8 fallback.

**Issue:** Creel's sweep moves a gone holder's unread message one step back to its sender and journals it undelivered only when that step is no strand (`internal/creelengine/doc.go`, Removed holders). A strand-signed notice lands in the sending strand's mailbox and is reminded there. D8's `--from <shortname>:BATTEN` fallback misses a landed notice from a batten run started in a strand pane (`internal/battenshed/land.go:407-418`).

### [LOW] The reply form is refused by the CLI

**Location:** `_lyx/discussion/decision-record.md` D4.

**Issue:** `post --reply` refuses anything but one text argument: "post --reply takes the text alone" (`internal/creelcli/cli.go:302-303`). The unquoted `written <path>` passes two.

### [LOW] The pause-resume line can expire

**Location:** `_lyx/discussion/decision-record.md` D9.

**Issue:** `queueResumeLine` marks the entry noticed right after queueing (`internal/orchengine/pause.go:232-247`). As a ten-minute inject the line expires if the orch stays busy, and is never sent again; the old notice had no age limit.

### [LOW] The revive inject's sender is unstated

**Location:** `_lyx/discussion/decision-record.md` D7.

**Issue:** `checkMayAct` refuses a non-privileged strand injecting into another strand (`internal/creelengine/verbs.go:456`). D7 names no sender, and D6's rule covers NotifyPrime callers only.

### [LOW] Concurrent revives both inject

**Location:** `_lyx/discussion/decision-record.md` D7.

**Issue:** The journal check and the inject are not atomic, so two `lyx tender revive` calls can each find no standing revive and each inject.

### [LOW] The revive report's claims go stale

**Location:** `_lyx/discussion/decision-record.md` Scope In.

**Issue:** `OrchRevive.Conditional` and `WatcherLive` describe watcher delivery (`internal/tenderengine/resume.go:264`, `:378`, `doc.go:55`); D7 changes the delivery and Scope's observable CLI list omits the report.

### [LOW] GUID to name is unstated

**Location:** `_lyx/discussion/decision-record.md` D6, D4.

**Issue:** `State.Strand` holds the GUID (`internal/orchengine/state.go`), while creel addresses and signs by `Strand.Name` (`internal/creelengine/seam.go`).

### [LOW] lyx orch status has no hold or send stuck fields

**Location:** `_lyx/discussion/decision-record.md` Scope In.

**Issue:** `statusFields` emits `stuck` only (`internal/orchcli/status.go:52`); `StuckByHold` and `StuckBySend` are read nowhere outside `internal/orchengine`. The status help text, "stuck names why an overdue phase is still waiting for the session to go idle", is what changes.

### [LOW] D2's stencil reason misreads the board's open point

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** The board's Open point says "the reminder is typed only at idle, but in Claude a typed line starts a new turn" and asks whether stencils should let agents leave a reminder. D2 answers that a reminder never interrupts a turn, which the question already grants.

## Rejected

- The Auto-mode assumption and the Open risk both naming the shortened PATTERN sentence: the open risk adds that parent review may restore it, so it is not a repeat.
- The new `PATTERN-durable-vs-ephemeral-state` text against D8's landing fallback reading the creel journal: the role starts only tasks the board still shows ready, and a missing line leaves them for the operator, so the journal stays a cache.
- D10's mailbox read at start is redundant, since a program's first post also makes its mailbox known: harmless, and no defect.
