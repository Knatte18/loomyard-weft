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
- The reminder line names only count, ids and senders.
- The orch watcher's cycle runs over creel: the note request is mail answered by post, the `/clear`, `/compact`, plugins, color, skills and pointer steps are command or text injects, and a creel hold keeps other deliveries off the orch during a cycle.
- The watcher's notice queue (`orchengine.QueueNotice`, `QueueConditionalNotice`, the `notices/` directory) and its idle probe before typing go.
- Every caller of the notice queue moves to creel: batten's notices, `orchcli.NotifyPrime` and its callers (burler, webster, loom driver hand-back, loom landing, shuttle's hold and dialog notices, hibernate), tender's orch revive and the watcher's own pause-resume line.
- The hub notice log (`.lyx/notices.log`, `shedrun.NoticeLog`, battenshed's `NoticeLog` seam) goes; the orch role's fallback reads creel instead.
- The orch stencils (role, handoff, soft handoff) and the package docs (`creelengine`, `creelcli`, `orchengine`, `orchcli`, `battenshed`, `tenderengine`) describe the new paths, including the observable CLI changes: `lyx creel inbox` loses its withheld flag, `lyx creel log` shows command injects, the reminder line changes, and `lyx orch status` loses its hold and send stuck fields.
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
Bound: on an orch strand, EXTERNAL senders, programs and other orch strands may now inject and interrupt, as on any strand; a hold and a command inject (D3) on an orch strand are refused from any sender but a program, so only Go code lyx ships can hold the orch or block its cycles.
Every such sender is a process of the operator's user, behind the 0700 lyxtemp root.
Rationale: the board scope says the withheld reminders of an orch strand are typed; the orch is the main receiver of stage 2's mail, and the watcher's cycle depends on a hold no one else can take.

### D2 — The reminder carries no content

The reminder reads `[creel] N unread: #id from <sender>, … — run lyx creel read`, at most five entries and `+K more`, with no subject.
Under a hold the reminder already lists only the owner's mail; its command then reads `lyx creel read --from <owner>`, so the receiver reads the held-for mail and not an older message from another sender.
This follows from the hold and adds no verb or flag.
The journal's `subject` field stays, and `lyx creel log` keeps showing it.
No stencil gains a line about when to answer a reminder: a reminder is typed only at idle, so it never interrupts a turn.
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
The body names the note path, the note template, the message id to reply to, and the two answers: `lyx creel post --reply <id> written <path>` once the note is written, or, for a soft request only, `lyx creel post --reply <id> defer`.
The handoff stencils lose their one-line constraint, since mail may span lines.
The watcher reads its own program mailbox each tick without waiting, and counts only a reply from the orch strand it is bound to whose `reply_to` is the pending request's id; every other message, a reply to an earlier request included, is read, logged and dropped.

- A body that is `written` followed by a path, whitespace trimmed, passes the note gate when the path, resolved against the prime, is the requested note path and the file exists and is non-empty.
  A reply naming another path, carrying other text, or arriving with a missing or empty file is logged and leaves the phase waiting until the handoff timeout, as an unwritten note does today.
- `defer` declines a soft cycle when the file is still unwritten, recording State.LastDeferral and returning to idle with `deferred`, as the `DEFER` turn end does today.
  In a hard or requested cycle a `defer` reply is logged and ignored.

The gates on the turn-end message (`DEFER`) and on a turn end read after the file was seen go: creel types the next command only once the turn that posted the reply has ended.
The abort on a session that asks a question during the handoff (an `EventWaiting` turn end) stays, read from the transcript.
Rationale: the board scope fixes the note request as mail answered by post; the reply carries the path, so the watcher knows which file it is told about.

### D5 — The watcher's cycle over creel

The phase machine, its triggers (hard, requested, soft), the trigger re-read, the timeouts and the persisted phases stay as they are; what changes is how each step is delivered and confirmed.

- A trigger fires on the existing gates minus the idle probe: a turn end is the newest event, the grace has passed, and the soft deferral hold allows it.
  Creel's own idle check, the same shuttle readiness reading, gates every delivery the cycle makes.
- Hold: at a cycle start the watcher takes a creel hold on the orch strand, renews it on every tick of the cycle, a tick held by a hub pause included, and releases it at every return to idle.
  A restarted watcher in a non-idle phase takes the hold again before anything else; a phase reset because it belongs to another strand releases that strand's hold and drops its pending ids.
  A creel refusal of the hold starts no cycle on that tick; the watcher logs it once per change and tries again.
  Bound: a watcher that dies mid-cycle leaves a hold that lapses within its duration, at most one hour; a live cycle keeps the orch's other mail unreminded until it returns to idle, which the existing phase timeouts bound everywhere but the clearing phase, where `lyx orch status` shows the stall.
- Each typed step is a creel inject: `/clear`, `/compact <focus>`, the plugins reload, the color and the skills load as command injects (D3); the resume pointer, the reload pointer and the pause-resume line as text injects.
- A step advances on creel's outcome for its inject id: `injected` advances; `expired` queues the same step again and records the stall reason creel reported in State.Stuck.
  `unconfirmed` means the line was typed and its landing not confirmed; it advances for `/clear`, as the old restart rule does, and for a step that waits on its own signal (a turn end or a compaction boundary), which keeps waiting until its timeout.
  Bound: this departs from the board's "confirms" for those steps only; an `/clear` that did not land leaves the resume pointer in an uncleared session, which the old watcher already accepted, and every other `unconfirmed` step is still gated by its own signal.
- Steps that wait on the session keep reading the transcript: the skills and retry steps classify their turn end, the pointer step ends the phase at its turn end, the compacting phase searches for its boundary, and the session-start hook's resume mark still lets the pointer step skip typing.
- Restart: before each post or inject the watcher persists the step as pending with no id, and saves the id creel answers once the call returns.
  A restarted watcher holding a pending step with no id looks it up in creel's journal (`log` for its own posts or injects to the orch since the phase began) and adopts the one it finds; only when none exists does it send the step.
  A restarted watcher holding an id reads its outcome and resumes without sending again.
- The watcher reads outcomes and replies from creel without blocking its tick.
- The color typed at bind is a `color` command inject; a failed or expired one is logged and dropped, as today.
- The auto-compaction reload entry decides as today; its boundary, held until the next passing probe today, is acted on at once by queueing the reload's first inject.

The watcher's idle probe before typing, the hold-reason bookkeeping (State.StuckByHold, State.StuckBySend), the `orch: injection held` logging, the conditioned-notice drop and the state-versus-probe disagreement log go.
State.Stuck holds the stall reason creel last reported for the watcher's pending inject, so `lyx orch status` still shows why a cycle waits.
A creel call that fails (daemon unreachable, a refusal) is logged once per change of error and is not counted toward the consecutive tick-error cap, so a creel outage holds the cycle and never ends the watcher.
Rationale: the board scope keeps only the cycle logic in the watcher and makes creel's confirmation the advance signal.

### D6 — Notices are creel mail to the orch

`orchcli.NotifyPrime` keeps its role as the one way a module notifies the prime's orch: it resolves the prime, reads the orch strand from the orch state and posts the line as creel mail to that strand.
With no orch strand recorded it logs the notice and posts nothing, as today.
A post refused or failed is returned to the caller, which warns as it does today; a notice stays informational and changes no control flow.
Batten's notice paths (`battencli`'s `Notify` deps and `drive.go`) post through the same function.
Sender: a `NotifyPrime` caller signs as its own strand when `$LYX_STRAND_NAME` names a strand that is not an orch, and otherwise as its module's program name, such as `BATTEN`, `TENDER` or `HIBERNATE`.
Each sending module owns its program name as a constant, formed through `agentname.FormatProgram` (`PATTERN-agent-name`).
This keeps creel's identity rules unchanged: a non-orch pane never claims a program name, and a batten run started from the orch's own pane signs as a program, so the orch never mails itself.
Notices keep their text, the `[batten notice]` prefix included, as the mail body.
The orch role's wake-up becomes a creel reminder: the orch reads its mail with `lyx creel read --all` and handles each `[batten notice]` body as the role says today.
Bound: unread mail to an orch strand that is removed for good (`lyx orch stop`, a relaunch that outlasts creel's sweep) returns to its program sender and is journaled undelivered; it stays readable in `lyx creel log`, which the role's fallback reads (D8).
Rationale: the board scope folds batten's notices and `QueueNotice` into creel; the notice queue goes, so every caller moves.

### D7 — Tender's orch revive is a checked text inject

Tender reads the orch session's state itself and, only while it reads `idle-stalled` with cause `api-error`, injects the revive line through creel with a TTL of at most one minute.
`lyx tender revive` keeps no memory between calls, so before injecting it asks creel's journal (`log`) for a revive inject of its own to the orch with no outcome yet, and injects none while one stands.
Bound: the condition is read before the inject instead of at typing, so a session that leaves the stall within that minute can still get the revive line typed, which starts one extra turn; the TTL caps that window.
Rationale: creel knows no session states, and the stalled session reads idle, so creel types the line within one probe interval.

### D8 — The hub notice log goes

Batten stops appending to `.lyx/notices.log`; `shedrun.NoticeLog`, `NoticeLogRel`, battenshed's `NoticeLog` seam and the help text naming the file go.
The orch role's landing fallback reads unread creel mail with `lyx creel read`, and for a notice already read or returned undelivered, its line in `lyx creel log --from <shortname>:BATTEN`, whose subject is a one-line notice's whole text.
`lyx orch cleanup --apply` removes the retired `notices/` directory and the retired `.lyx/notices.log` in the prime.
Rationale: a creel notice stays unread until the orch reads it, and the journal holds it after, so the log is a second copy.

### D9 — The hub pause stays in the watcher

The watcher's hub-pause handling keeps `Session.SessionIdle` and shuttle's guarded interrupt, unchanged; `SessionIdle` stays in the `Session` seam for this handler only.
Creel refuses an interrupt under a covering pause, and the pause is exactly when this interrupt is sent.
The pause-resume line it queued as a notice becomes a text inject (D5), which creel types once the pause is lifted.
Creel's pause coverage already holds every delivery into the orch under a hub-wide entry, so no notice reaches the orch during a pause.
Rationale: the pause interrupt is a key, not a delivery, and `PATTERN-guarded-interrupt` already names it as the one exemption.

### D10 — The watcher reaches creel through a seam

`internal/orchengine` gets a creel seam beside its `Session` seam (post, read mailbox, inject, outcome, journal lookup, hold, release), and `internal/orchcli` wires it over creel's Go client.
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
- A watcher restarted in every non-idle phase, with the pending step's id saved or not yet saved, resumes without posting or injecting a second time and takes the hold again.
- A cycle spanning a hub pause keeps its hold; a phase reset for another strand releases the old hold.
- An expired step inject is queued again and the stall reason shows in `lyx orch status`.
- A creel call failure holds the cycle, logs once per change and never counts toward the tick-error cap.
- Under a hub pause the watcher's interrupt behaves as today, and the pause-resume line reaches the orch as an inject after the pause lifts.
- A batten notice, a `NotifyPrime` notice and a hibernate notice arrive as creel mail to the orch strand, signed by the caller's non-orch strand, else by its program name, also when the caller runs in the orch's own pane; with no orch strand recorded nothing is posted and the notice is logged.
- Mail to an orch strand removed for good is journaled undelivered and shown by `lyx creel log`.
- Tender injects a revive only while the orch reads `idle-stalled` with cause `api-error`, with a TTL of at most one minute, and never while its earlier revive has no outcome.
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
