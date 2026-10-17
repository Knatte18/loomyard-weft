# Decision record — creel-notices

## Goal

Creel stage 2: every notice and every line lyx types into the orch session goes through creel.
The orch watcher keeps only its cycle logic: tracking tokens, choosing a `refresh`, `distill` or automatic cycle, waiting for the note, sequencing the reload, its state and its lifecycle.
Its notice queue and its idle probe before typing go.
Creel's reminder stops carrying message content.

## Scope

In:

- Creel types into orch strands: reminders, injects and the new command injects; the stage 1 withholding goes.
- Command injects in creel: a closed set of provider session commands a program sender queues for an orch strand, typed by creel through shuttle's session operations.
- A Go-client-only creel `withdraw` verb, with which a program removes its own queued injects and its own unread mail to one receiver, used when the orch is relaunched and when the watcher resets a phase that belongs to another strand (D5).
- The reminder line names only count, ids and senders.
- A shuttle change that gives each event its hook-side time, which the watcher's reload steps compare with creel's `sending` time (D5):
  `shuttleengine.Event` (`internal/shuttleengine/engine.go`) gains a time field;
  `Runner.ReadEvents` (`internal/shuttleengine/session.go`) sets it from the engine's `ParseSessionSignals` and resumes from that reading's consumed count;
  the fake engine in `internal/testkit/shuttlefake/shuttlefake.go` gains a `ParseSessionSignals` that gives each Stop line a time, so a cycle test against it can confirm a step.
  The orchengine `Session` seam passes the field through unchanged; the claude `ParseEvents` and the other `ParseEvents` callers stay as they are, and only `Runner.ReadEvents` sets the field.
- A typed refusal code on creel's response (`internal/creelengine/protocol.go`), set only on the refusal of a strand claim from a process in no strand pane, which the Go client (`internal/creelengine/client.go`) returns as an exported error value that `NotifyPrime` matches with `errors.Is` (D6).
- The orch watcher's cycle runs over creel: the note request is mail answered by post, the `/clear`, `/compact`, plugins, color, skills and pointer steps are command or text injects, and a creel hold keeps other deliveries off the orch during a cycle.
- The watcher's notice queue (`orchengine.QueueNotice`, `QueueConditionalNotice`, the `notices/` directory) and its idle probe before typing go.
- Every caller of the notice queue moves to creel: batten's notices, `orchcli.NotifyPrime` and its callers (burler, webster, loom driver hand-back, loom landing, shuttle's hold and dialog notices, hibernate), tender's orch revive and the watcher's own pause-resume line.
- The hub notice log (`.lyx/notices.log`, `shedrun.NoticeLog`, battenshed's `NoticeLog` seam) goes; the orch role's fallback reads creel instead.
- The orch stencils (role, handoff, soft handoff) and the package docs (`creelengine`, `creelcli`, `orchengine`, `orchcli`, `battenshed`, `tenderengine`, `shuttleengine`) describe the new paths, creel's typed refusal code in the `creelengine` and `creelcli` docs included, and the observable CLI changes: `lyx creel inbox` loses its withheld flag, `lyx creel log` shows command injects, the reminder line changes, the orch state loses its `stuck_by_hold` and `stuck_by_send` keys and gains the record of a failed second fresh-launch withdraw or a failed release (D5), `lyx orch start` fails and launches no session when its first withdraw fails (D5), `lyx orch status`'s help text says `stuck` names the stall reason creel last reported for the watcher's pending inject or the creel failure holding the cycle, and `lyx tender revive`'s report for the orch drops its conditional-delivery and watcher-live fields and reports the inject id (D7).
- The `PATTERN.md` changes listed below, with their background files.

Out:

- The wording of the notices and of shuttle's hold notice, and the de-duplication of drivers' stop reports against batten's notices: stage 3, `creel-agents`. This task changes how notices travel, not what they say.
- Agents messaging each other, the parent-review gate, `lyx shuttle send` and `lyx shuttle interrupt`, the reply-discipline rules and reply chasing: stage 3.
- Direct pane typing into strands other than the orch (shuttle send, tender's revive of non-orch strands): stage 3 or unchanged.
- The hub-pause handling: it keeps the watcher's `Session.SessionIdle` readiness reading and the guarded interrupt (D9), a bounded departure from "the idle probe goes".
- Who in a running task receives an orch instruction (`orch-split`'s routing question).
- Cross-machine delivery of the creel journal.

## Decisions

### D1 — Creel delivers into orch strands

Creel treats an orch strand as a receiver like any other for mail and injects: its reminders are typed, and the stage 1 refusal and `withheld` record go.
`inbox` loses its withheld flag.
A journal from stage 1 holding `withheld` records still folds: the fold reads them and acts on none (`PATTERN-tolerant-records`).
Mail to an orch strand left unread from stage 1 is reminded from the first probe after the deploy.
Bound: on an orch strand, EXTERNAL senders, programs and other orch strands may now inject and interrupt, as on any strand; a hold and a command inject (D3) on an orch strand are refused from any sender but a program, so only Go code lyx ships can hold the orch.
A cycle can also be blocked by creel's sender resolution: while any live strand's pane process is unknown or any worktree's reed state is unreadable, creel refuses every sender outside a pane, the watcher included, and D5 bounds that case.
Every such sender is a process of the operator's user, behind the 0700 lyxtemp root.
Rationale: the board scope says the withheld reminders of an orch strand are typed; the orch is the main receiver of stage 2's mail, and the watcher's cycle depends on a hold no one else can take.

### D2 — The reminder carries no content

The reminder reads `[creel] N unread: #id from <sender>, … — run lyx creel read`, at most five entries and `+K more`, with no subject.
Under a hold the reminder already lists only the owner's mail; its command then reads `lyx creel read --from <owner>`, so the receiver reads the held-for mail and not an older message from another sender.
This follows from the hold and adds no verb or flag.
The journal's `subject` field stays, and `lyx creel log` keeps showing it.
No stencil gains a line letting an agent leave a reminder for later: when an agent reads its mail is part of stage 3's reply-discipline rules, which Scope Out leaves to `creel-agents`.
Rationale: operator decision of 2026-10-17 on the board entry; the `--from` command keeps the orch's note request from being shadowed by older notices.

### D3 — Command injects

A creel inject carries either a text line, as today, or a command from a closed set: `clear`, `compact` with a one-line focus, `reload-plugins`, `color` with a palette color, and `load-skills` with a skill list.
The creel engine knows only the neutral command name and its argument; no slash spelling or key name appears in `internal/creelengine`.
Its `Delivery` seam gains one operation that runs a command on a strand, which `internal/creelcli` implements over shuttle's session operations (`ClearSession`, `CompactSession`, `ReloadPlugins`, `TypeColor`, `LoadSkills`), so provider spellings stay in shuttle (`PATTERN-shuttle-provider-seam`).
A command inject follows every inject rule: queued, typed only when the receiver is idle and unpaused, ordered with other injects, held by a hold, kept for a dormant receiver, journaled `sending` before typing and `injected`, `unconfirmed` or `expired` after, never typed twice.
A shuttle refusal that typed nothing (`ErrSessionBusy`) requeues it like a busy text send.
Bound: only a program sender, through the Go client, may queue a command inject, and only for an orch strand; the CLI has no flag for one, and any other sender or receiver is refused.
A command inject can therefore clear, compact or reload only the orch session, only from Go code lyx ships, and only when creel's idle check passes.
`lyx creel log` shows a command inject by its command name and argument.
Rationale: the board scope says a cycle phase advances "once creel confirms a command inject was typed"; shuttle's session operations are key sequences, not one text line, so the inject has to name the operation.

### D4 — The note request is mail, answered by post

To request a note, the watcher posts mail to the orch strand, with the rendered handoff or soft-handoff stencil as body.
The body names the note path, the note template, the message id to reply to, and the two answers: `lyx creel post --reply <id> "written <path>"` once the note is written, or, for a soft request only, `lyx creel post --reply <id> defer`.
The handoff stencils lose their one-line constraint, since mail may span lines.
The watcher reads its own program mailbox each tick without waiting, and counts only a reply from the orch strand it is bound to whose `reply_to` is the pending request's id; every other message, a reply to an earlier request included, is read, logged and dropped.
The orch strand's name is resolved from the GUID the orch state records through reed's state at each call, and the sender check compares the reply's sender with that resolved name.
The watcher persists the gate's outcome, the move to clearing or the deferral, inside the Go client's read handler, which runs before the client acknowledges the read, so a watcher that dies before the save leaves the reply unread for its restart.

- A body that is `written` followed by a path, whitespace trimmed, passes the note gate when the path, resolved against the prime, is the requested note path and the file exists and is non-empty.
  A reply naming another path, carrying other text, or arriving with a missing or empty file is logged and leaves the phase waiting until the handoff timeout, as an unwritten note does today.
- `defer` declines a soft cycle when the file is still unwritten, recording State.LastDeferral and returning to idle with `deferred`, as the `DEFER` turn end does today.
  In a hard or requested cycle a `defer` reply is logged and ignored.

The gates on the turn-end message (`DEFER`) and on a turn end read after the file was seen go: creel types the next command only once the turn that posted the reply has ended.
The abort on a session that asks a question during the handoff (an `EventWaiting` turn end) stays, read from the run's events file.
Rationale: the board scope fixes the note request as mail answered by post; the reply carries the path, so the watcher knows which file it is told about.

### D5 — The watcher's cycle over creel

The phase machine, its triggers (hard, requested, soft), the trigger re-read, the timeouts and the persisted phases stay as they are, but for the reload steps' timeouts and the pending step's persist time below; what changes is how each step is delivered and confirmed.

- A trigger fires on the existing gates minus the idle probe: a turn end is the newest event, the grace has passed, and the soft deferral hold allows it.
  Creel's own idle check, the same shuttle readiness reading, gates every delivery the cycle makes.
- Hold: on entering any non-idle phase, a cycle start and the auto-compaction reload's resuming phase alike, the watcher takes a creel hold on the orch strand, renews it on every tick outside idle, a tick held by a hub pause included, and releases it at every return to idle.
  A restarted watcher in a non-idle phase takes the hold again before anything else; a phase reset because it belongs to another strand withdraws ORCHWATCH's queued injects and unread mail to the orch strand, releases its hold and drops its pending ids.
  A creel refusal of the hold starts no cycle on that tick; the watcher logs it once per change and tries again.
  A refusal from creel's sender resolution (D1) is that case too: the cycle waits, and `lyx orch status` shows the refusal as State.Stuck until creel resolves the watcher again.
  Bound: a watcher that dies mid-cycle leaves a hold that lapses within its duration, at most one hour; a live cycle keeps the orch's other mail unreminded until it returns to idle, which the phase timeouts, the color and plugins steps' timeout below included, bound everywhere but the clearing phase, where `lyx orch status` shows the stall.
- Fresh launch: `lyx orch start`, wherever it calls `ResetForFreshLaunch` (a relaunch, `--continue`, `--adopt` and an adopted strand), withdraws every inject ORCHWATCH has queued for the orch strand and releases ORCHWATCH's hold on it, signing as `ORCHWATCH` through the Go client.
  It does so twice: before it starts or binds the session, so no old inject is typed into a session that comes up idle, and again after it saves the reset state, which catches a step an old watcher still running queued in between.
  Creel gains the verb for it, `withdraw`, Go client only: a program sender removes its own queued injects and its own unread mail to one receiver, each journaled with reason `withdrawn`, an inject as `expired` and a message as `undelivered`.
  `withdraw` acts on the receiver by name whether or not creel still lists that strand, as `release` does for a hold the caller still has there, and answers an empty result when nothing of the caller's is queued there, so a continue or adopt after `lyx orch stop`, which leaves no orch strand listed, passes it.
  `lyx orch start` runs the first withdraw before it removes the old strand.
  When no orch strand is listed, `lyx orch start` sends no first release, since creel refuses a release to an unlisted name the caller holds no hold on; the second release, sent once the new strand under the same fixed name is listed, ends any hold ORCHWATCH still has there.
  A watcher that reads `withdrawn` for its pending inject queues nothing again and returns to idle, releasing its hold, and a watcher queues a step only after re-reading the state file on that tick and finding its phase still non-idle on its strand, so an old watcher stops at the reset state.
  Bound: `withdraw` touches only the caller's own queued injects and unread mail; an inject already journaled `sending` is typed to the end, and a message the orch already read stays read, so at most one line of the old cycle reaches the relaunched session.
  This bound holds once both withdraws have succeeded, but for the one step per old watcher that the race in the bound below admits; that bound covers a failed second withdraw and a failed release.
  `withdraw` answers the id and subject of each message it removed, and the orchengine creel seam's withdraw returns them; whichever caller's withdraw removes a pause-resume line (D9), `lyx orch start` or the watcher in its retried withdraw or its phase-reset withdraw, clears the owning pause entry's resume-noticed mark under the record lock.
  A continued or adopted session, whose GUID the entry still names, then gets the line posted again by the watcher, and a relaunched session, whose GUID differs, gets none, since it never saw the pause.
  When the first withdraw fails, a refusal or an unreachable daemon alike, `lyx orch start` starts and binds no session and fails with creel's error and the way forward: repair what the error names, such as a worktree whose reed state the sender-resolution refusal names as unreadable, or a daemon that would not start, whose log lines the Go client's connect failure quotes, then run `lyx orch start` again.
  Cost: while that cause stands, no orch can be started, continued or adopted; the alternative, a session an old `/clear` can reach, was rejected.
  When the second withdraw or either release fails, `lyx orch start` records that in the orch state, and the watcher it spawns retries both, signing as `ORCHWATCH`, on every tick before it posts or queues anything, clearing the record once both succeed; until then it posts and queues nothing, the bind color included, starts no cycle and shows the failure as State.Stuck.
  Every step inject the watcher queues carries a TTL of ten minutes, creel's default inject TTL, named in the watcher's code.
  Bound: while the second withdraw has failed and no retry has succeeded, an old ORCHWATCH inject queued between the two withdraws, a `/clear` or `/compact` included, is typed on the relaunched session's first idle, unpaused probe if that probe comes within the ten-minute TTL, which runs while the session is busy; otherwise it expires;
  and old ORCHWATCH mail posted in that window, a note request or a pause-resume line, is reminded to the relaunched orch until a retry succeeds, since mail has no TTL.
  While a release has failed and no retry has succeeded, ORCHWATCH's hold stays on the relaunched orch, so creel reminds only ORCHWATCH's mail and no other sender's, until a retry succeeds or the hold lapses within the Hold bullet's one hour; the watcher, idle after the reset, renews no hold.
  This exposure is accepted.
  Its step part does not need a failed withdraw: an old watcher queues a step only after re-reading the state file, so each old watcher still running sends at most the one step of a tick that read the state before the reset was saved, and a successful second withdraw removes that step only when it was queued first.
  The pause-resume line, which an old watcher posts with no such re-read, can arrive at any time an old watcher runs, and is informational to a session that never saw the pause.
- Each typed step is a creel inject: `/clear`, `/compact <focus>`, the plugins reload, the color and the skills load as command injects (D3); the resume pointer and the reload pointer as text injects.
- A step advances on creel's outcome for its inject id: `injected` advances; `expired` queues the same step again and records the stall reason creel reported in State.Stuck.
  `unconfirmed` means the line was typed and its landing not confirmed; it advances for `/clear`, as the old restart rule does, for the color and plugins steps and the bind color, which have no confirmation today, and for a step that waits on its own signal (a turn end or a compaction boundary), which keeps waiting until its timeout.
  Bound: this departs from the board's "confirms" for those steps only; an `/clear` that did not land leaves the resume pointer in an uncleared session, which the old watcher already accepted, a color or plugins step that did not land leaves that step undone, as a timed-out one does, and every other `unconfirmed` step is still gated by its own signal.
- The color and plugins steps get the handoff timeout; past it the reload logs the step as skipped and moves on, as a skills step past its timeout does.
  Every reload step's timeout counts from its first queueing instead of its first typing, so a step whose inject keeps expiring and is never typed still times out.
- Steps that wait on the session keep reading the run's events file for turn ends and the transcript for what a turn did: the skills and retry steps classify their turn end, the pointer step ends the phase at its turn end, the compacting phase searches for its boundary, and the session-start hook's resume mark still lets the pointer step skip typing.
  Such a step reads the run's events file, and counts a turn end only when its stamp's second is after the second of creel's `sending` record for its inject, whose time the watcher reads from the journal with the inject's outcome, so a turn end from the queueing through the second of `sending` confirms nothing.
  `Runner.ReadEvents` gives each event the hook-side time of its line from the engine's session-signal reading (`ParseSessionSignals`), pairing in file order the events with the signals that mark a line `ParseEvents` also yields, and an engine with no such reading gives every event the zero time.
  It advances its offset only through that reading's consumed count, which stops before a trailing run of stamp lines with no payload after them, so a stamp read in one tick and its Stop payload in the next are read together on the next tick.
  That reading's stamp rule, under which a payload with no preceding stamp or a malformed one reads the zero time, is the one place that defines a Stop with no readable stamp, and such a Stop never counts.
  Bound: the stamp has whole-second precision, so a step whose own turn ends within the second it was typed is missed and times out, as an unconfirmed step does, and another turn that ends in the second after `sending`, before the typing finishes, still counts, a window creel's idle check before `sending` keeps narrow.
- Restart: before each post or inject the watcher persists the step as pending with no id, with the time it persisted it, and saves the id creel answers once the call returns.
  The step's first queueing time, from which its timeout counts, is persisted at its first queueing and kept across every requeue and restart.
  A restarted watcher holding a pending step with no id looks it up in creel's journal (`log` for its own posts or injects to the orch since the pending step was persisted) and adopts only one whose command name and argument, or text, match the pending step; only when none exists does it send the step.
  The persist time is what tells the skills and retry steps apart, which can carry the same `load-skills` argument: the retry step is persisted only after the skills inject's outcome is read.
  A restarted watcher holding an id reads its outcome and resumes without sending again.
- The watcher reads outcomes and replies from creel without blocking its tick.
- The color typed at bind is a `color` command inject; a failed or expired one is logged and dropped, as today, and an `unconfirmed` one counts as typed.
- The auto-compaction reload entry decides as today; its boundary, held until the next passing probe today, is acted on at once by queueing the reload's first inject.

The watcher's idle probe before typing, the hold-reason bookkeeping (State.StuckByHold, State.StuckBySend), the `orch: injection held` logging, the conditioned-notice drop and the state-versus-probe disagreement log go.
State.Stuck holds the stall reason creel last reported for the watcher's pending inject, or the creel failure or refusal holding the cycle, so `lyx orch status` still shows why a cycle waits.
A creel call that fails (daemon unreachable, a refusal, the sender-resolution refusal included) is logged once per change of error, shown as State.Stuck, and is not counted toward the consecutive tick-error cap, so a creel outage holds the cycle and never ends the watcher.
Rationale: the board scope keeps only the cycle logic in the watcher and makes creel's confirmation the advance signal.

### D6 — Notices are creel mail to the orch

`orchcli.NotifyPrime` keeps its role as the one way a module notifies the prime's orch: it resolves the prime, reads the orch strand's GUID from the orch state, resolves its name through reed's state at each call and posts the line as creel mail to that strand.
With no orch strand recorded it logs the notice and posts nothing, as today.
A post refused or failed is returned to the caller, which warns as it does today; a notice stays informational and changes no control flow.
Batten's notice paths (`battencli`'s `Notify` deps and `drive.go`) post through the same function.
Sender: when `$LYX_STRAND_NAME` names a strand that is not an orch, `NotifyPrime` posts with that strand claim; when creel refuses it because the process runs in no strand pane, it posts once more as its module's program name with no strand claim; in every other case it posts as its program name, such as `BATTEN`, `TENDER` or `HIBERNATE`, with no strand claim, as D10 does for the watcher.
The caller walks no process ancestry of its own, so its choice never disagrees with creel's sender resolution.
It recognises that refusal by creel's typed refusal code: the creel response carries the code only on the refusal of a strand claim from a process in no strand pane, and the Go client returns it as an exported error value that `NotifyPrime` matches with `errors.Is`, never by the message text.
Bound: the program retry happens only on that refusal, so a caller in a non-orch pane never signs as a program.
Creel gives that refusal only for a strand it lists; a claim naming a strand creel no longer lists, such as a removed task strand a detached loom driver outlived, is signed EXTERNAL and posted as such.
A claim naming one strand from another strand's pane is refused with no retry, and the notice is warned and dropped.
A caller in a non-orch pane whose environment carries no strand claim posts as its program, is refused because only an orch strand's pane may claim a program name, gets no retry, and the notice is warned and dropped.
Creel's refusal of every sender outside a pane while it cannot resolve senders (D1) gets no retry, and the notice is warned and dropped as any refused post is.
A dropped notice survives only in the caller's warning: the error `NotifyPrime` returns carries the notice's full line and creel's refusal, so whichever log or pane the caller warns into, a detached caller's included, holds the line for the operator to recover.
A detached caller, such as `hibernate watch` or a detached loom driver, inherits the spawning pane's `$LYX_STRAND_NAME`.
On Linux, while its spawning parent lives, its ancestry still reaches the pane and it signs as the strand; once that parent has exited and while creel still lists the strand, creel refuses the strand claim and it signs as its program.
Elsewhere creel takes identity from the environment alone, so it signs as the strand throughout.
Each sending module owns its program name as a constant, formed through `agentname.FormatProgram` (`PATTERN-agent-name`).
This keeps creel's identity rules unchanged: a non-orch pane never claims a program name, and a batten run started from the orch's own pane signs as a program, so the orch never mails itself.
Notices keep their text, the `[batten notice]` prefix included, as the mail body.
The orch role's wake-up becomes a creel reminder: the orch reads its mail with the command the reminder names, `lyx creel read --from <owner>` under a hold (D2) and `lyx creel read --all` otherwise, and handles each `[batten notice]` body as the role says today; the role stencil says so.
Bound: unread mail to an orch strand that is removed for good (`lyx orch stop`, a relaunch that outlasts creel's sweep) goes one step back to its sender.
A program-signed notice is journaled undelivered; a strand-signed one returns to the sending strand's mailbox and is reminded there.
Either way its post stays in `lyx creel log`, which the role's fallback reads (D8).
Rationale: the board scope folds batten's notices and `QueueNotice` into creel; the notice queue goes, so every caller moves.

### D7 — Tender's orch revive is a checked text inject

Tender reads the orch session's state itself and, only while it reads `idle-stalled` with cause `api-error`, injects the revive line through creel with a TTL of at most one minute.
It injects as its program, `TENDER`, with no strand claim.
The orch revive runs only in the hub scope, which `tendercli` already refuses to an agent's pane with the way forward of asking its parent, so the `TENDER` claim comes from the orch's pane or from outside any strand pane, both of which creel accepts.
`lyx tender revive` keeps no memory between calls, so before injecting it asks creel's journal (`log`) for a revive inject of its own to the orch with no outcome yet, and injects none while one stands.
The orch revive, which today reads the record outside any lock, moves its lookup and inject into a `pauserecord.Update` of its own, the lock that serialises tender's task-session revive sends, so two concurrent revives inject at most once.
Bound: a pause, resume or watcher record write then also waits at most one revive's creel lookup and inject, each capped by the Go client's call timeout, and `tenderengine`'s doc restates its lock bound to say so.
The report for the orch drops its conditional-delivery and watcher-live fields and reports the inject id, and `tenderengine`'s doc says so.
Bound: the condition is read before the inject instead of at typing, so a session that leaves the stall within that minute can still get the revive line typed, which starts one extra turn; the TTL caps that window.
Rationale: creel knows no session states, and the stalled session reads idle, so creel types the line within one probe interval.

### D8 — The hub notice log goes

Batten stops appending to `.lyx/notices.log`; `shedrun.NoticeLog`, `NoticeLogRel`, battenshed's `NoticeLog` seam and the help text naming the file go.
The orch role's landing fallback reads unread creel mail with `lyx creel read`, and for a notice already read or returned undelivered, its line in `lyx creel log --to <orch strand>`, whose subject is a one-line notice's whole text; the `--to` filter finds a notice whichever sender signed it (D6).
A notice creel refused is not in `lyx creel log`; it survives only in its caller's warning (D6).
`lyx orch cleanup --apply` removes the retired `notices/` directory and the retired `.lyx/notices.log` in the prime.
Rationale: a creel notice stays unread until the orch reads it, and the journal holds it after, so the log is a second copy.

### D9 — The hub pause stays in the watcher

The watcher's hub-pause handling keeps `Session.SessionIdle` and shuttle's guarded interrupt, unchanged; `SessionIdle` stays in the `Session` seam for this handler only.
Creel refuses an interrupt under a covering pause, and the pause is exactly when this interrupt is sent.
The pause-resume line it queued as a notice becomes creel mail from `ORCHWATCH` to the orch strand, which creel reminds once the pause is lifted; mail has no TTL, so the line cannot expire while the orch stays busy.
The watcher marks the entry noticed only after the post succeeds, so a failed post is posted again on the next tick.
A line the orch left unread when the orch is relaunched, or when a phase is reset for another strand, is withdrawn, and posted again only to a continued or adopted session, whether `lyx orch start` withdrew it or the watcher did in its retried withdraw or its phase-reset withdraw (D5); while a relaunch's second withdraw has failed, a line posted between the two withdraws is still reminded, as D5 bounds.
Creel's pause coverage already holds every delivery into the orch under a hub-wide entry, so no notice reaches the orch during a pause.
Rationale: the pause interrupt is a key, not a delivery, and `PATTERN-guarded-interrupt` already names it as the one exemption.

### D10 — The watcher reaches creel through a seam

`internal/orchengine` gets a creel seam beside its `Session` seam (post, read mailbox, inject, outcome, journal lookup, hold, release, withdraw), and `internal/orchcli` wires it over creel's Go client.
The watcher always signs as its own program, `ORCHWATCH`, whatever `$LYX_STRAND_NAME` it inherited from the shell that ran `lyx orch start`; D6's strand rule is for `NotifyPrime` callers only.
The watcher reads its mailbox once at start, so creel knows the program as a receiver before a reply can arrive.
`internal/orchengine` imports no creel package (`PATTERN-told-geometry`), and its tests drive the cycle against a fake.
`internal/creelcli` exposes the Go-side calls other modules need, so no caller builds a creel request by hand.
Rationale: keeps the engine told and testable, as the `Session` seam does.

## Constraints

- `PATTERN-creel-store`: only the creel daemon writes the journal and every keystroke goes through the `Delivery` seam; the command operation joins that seam, and `internal/creelengine` still imports neither reed nor shuttle.
- `PATTERN-shuttle-provider-seam`: command names are provider-neutral; each provider's keys stay in shuttle's session operations.
- `PATTERN-tolerant-records`: new journal fields (the command name and argument) and old `withheld` records never fail a fold.
- `PATTERN-daemon-wakeups`: the watcher's per-tick creel calls ride its existing poll and add no wake of their own.
- `PATTERN-told-geometry`: orchengine stays told, and its import allowlist in `internal/orchengine/seam_enforcement_test.go` gains no creel package; the creel geometry comes from `hubgeom.CreelGeometry` in the wiring.
- `PATTERN-agent-name`: every program name goes through `agentname.FormatProgram`.
- `PATTERN-no-denied-recovery`: the stencils name only `lyx creel` verbs the orch's settings allow.
- Creel's identity rules are unchanged (D6).

## Auto-mode assumptions

- The orch's recommendation to remove the notice log is taken (D8).
- The operator's PATTERN sentence is taken with "such as the creel journal and the notice log" shortened to "such as the creel journal", because D8 removes the notice log in the same commit; the full shortened text is under `## PATTERN changes`.

## Open risks

- Whether shuttle's verified session operations report a landed `/clear`, `/compact` or `/reload-plugins`, which start no turn, as landed or as `ErrNotLanded`; an acceptance criterion pins it, and D5 bounds the `unconfirmed` case.
- The note request costs the orch two hops, a reminder turn and a read, where the old typed instruction cost one turn; a hard cycle at the context cap gets one more turn of context use before the note.
- Notice files queued under `.lyx/orch/notices/` at the deploy are not delivered.
- The shortened operator PATTERN sentence departs from the operator's wording; parent review may restore it.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - withdraw-unlisted-receiver-unstated (class decision, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The reminder line holds no subject; under a hold it lists only the owner's mail and names `lyx creel read --from <owner>`.
- An orch strand gets reminders and text injects; mail left unread from stage 1 is reminded; a journal with `withheld` records folds.
- A hold or command inject on an orch strand from EXTERNAL or a strand is refused; a program's succeeds.
- A command inject from a program to an orch strand is typed through the seam's command operation and journaled `sending`, then `injected`, `unconfirmed` or `expired`; a command inject to a non-orch strand is refused; a busy refusal requeues it; a hold and a pause hold it.
- Each command reaches the matching shuttle session operation through `creelcli`'s delivery, and the outcome shuttle reports for each command on a live Claude pane is pinned.
- A clear cycle runs end to end against a fake creel: hold taken, note request posted, `written <path>` reply passes the gate, `/clear` injected, the reload injects in order, the pointer's turn end ends the phase, hold released.
- A compact cycle runs the same way through `/compact <focus>` and the boundary search, and times out to `compaction timed out` when no boundary appears.
- A soft cycle's `defer` reply records the deferral and returns to idle; a hard cycle ignores `defer`.
- A `written` reply naming another path, carrying other text, or with a missing or empty note file never reaches `/clear`; a reply to an earlier request id or from another sender is dropped.
- A watcher restarted in every non-idle phase, with the pending step's id saved or not yet saved, resumes without posting or injecting a second time and takes the hold again; in the resuming phase this holds at each reload step, the pointer and a retry of every skill included, and the restart adopts that step's own inject, never an earlier step's.
- A watcher that dies after reading a `written <path>` reply and before saving the clearing phase finds the reply unread after its restart and gates on it.
- An orch relaunched, continued or adopted mid-cycle whose withdraws both succeed, with no old watcher queueing after the second withdraw, gets none of the old cycle's queued injects and is reminded of no note request from the old cycle, and its other mail is reminded at once.
- A relaunch whose first withdraw is refused or cannot reach creel starts and binds no session and reports creel's error with the way forward.
- A continue or adopt launched after `lyx orch stop`, with the orch strand removed and nothing queued, passes the first withdraw, sends no first release, starts the session and records no fresh-launch failure.
- A relaunch whose second withdraw or a release fails, refused or unable to reach creel, gets both retried by the watcher, which posts and queues nothing until a retry succeeds, and no old `/clear` reaches the session once a retry succeeds; the watcher renews no hold while idle with a failed release, so the hold lapses.
- A cycle spanning a hub pause keeps its hold; a phase reset for another strand releases the old hold; the auto-compaction reload holds the orch through its reload steps.
- An expired step inject is queued again and the stall reason shows in `lyx orch status`.
- An `unconfirmed` color or plugins inject advances the reload; a plugins inject that keeps expiring is skipped at the handoff timeout from its first queueing.
- An operator turn that ends between a step's queueing and its `sending` record confirms neither the skills, the retry nor the pointer step, and a turn end stamped in the second of the `sending` record confirms nothing.
- A step's Stop whose stamp line is read in one tick and whose payload is read in the next still counts.
- A creel call failure holds the cycle, logs once per change and never counts toward the tick-error cap; a sender-resolution refusal of the hold holds the cycle and shows in `lyx orch status` as `stuck`.
- Under a hub pause the watcher's interrupt behaves as today, and the pause-resume line reaches the orch as mail reminded after the pause lifts, also when the orch stays busy past the inject TTL.
- A batten notice, a `NotifyPrime` notice and a hibernate notice arrive as creel mail to the orch strand, signed by the caller's non-orch strand when creel accepts that claim, by its program name after creel refuses the claim as running in no strand pane, a detached caller on Linux whose spawning parent has exited included, as EXTERNAL when the claimed strand is no longer listed, and by its program name otherwise, also when the caller runs in the orch's own pane and when a detached `hibernate watch` was spawned from the orch's pane; with no orch strand recorded nothing is posted and the notice is logged.
- Mail to an orch strand removed for good is journaled undelivered when program-signed and returned to the sending strand when strand-signed, and either is shown by `lyx creel log --to <orch strand>`.
- Tender injects a revive as `TENDER` only while the orch reads `idle-stalled` with cause `api-error`, with a TTL of at most one minute, and never while its earlier revive has no outcome, two concurrent revives included; its report names the inject id.
- No code writes `.lyx/notices.log` or `.lyx/orch/notices/`, and `lyx orch cleanup --apply` removes both when present.

## PATTERN changes

- `PATTERN-orch-pane-single-writer` — changes: Typing into the orch session from Go: only creel, through its `Delivery` seam; the orch watcher keeps only a hub pause's guarded interrupt, reed's `/rename` repair keeps its own guard, and other modules post or inject through creel.
- `PATTERN-durable-vs-ephemeral-state` — changes: Adding a file: tracked under `_lyx`, never-tracked under `.lyx`, same subpath; nothing a run, the orch or a landing depends on lives only under `.lyx`. The background file gains, verbatim: "Nothing a run, the orch or a landing depends on lives only under `.lyx`: decisions, landings, stop reports and board state are always in git (board, records, decision record), and never-tracked state such as the creel journal is a delivery path or cache that a fresh machine starts without."

## Notes for the plan writer

- `pattern/PATTERN-guarded-interrupt.md` names the single-writer exemption and needs its wording aligned with the changed entry.
- `internal/creelengine/doc.go`'s "Unbuilt: stages 2 and 3" section is rewritten to match the scoped `PATTERN-orch-pane-single-writer` text.
- The watcher's outcome and journal lookups can use creel's `log --id` and `log --from` queries, or a resumable `wait-inject` in a goroutine.
- `cmd/lyx/testdata/test-budget.yaml` likely moves for `internal/orchengine` and `internal/creelengine`.
- `internal/testkit/envkit/envkit.go` lists the `NoticeLog.Now` fields that D8 removes.
