# Decision record — creel-stage1-fixes

## Goal

Fix the eight defects the creel stage 1 audit filed (#657 to #664) as each issue's Expected section says, so that creel reaches every hand-added strand or says why it cannot, loses no mail, and prints consistent output.
Make the audit's proposed sandbox scenarios C1 to C13 re-runnable by writing them into the shuttle sandbox suite beside S9.

## Scope

In:

- D1 to D8, one per issue, and D9's sandbox scenarios.
- D1 changes the input-box reading of every Claude run's send path, not only hand-added strands.
- The docs these change, in the same commit: `internal/creelengine/doc.go`, `internal/creelcli/doc.go`, `internal/shuttleengine/doc.go`, `internal/reedengine/doc.go`, the claudeengine comments that state the placeholder residual, `pattern/PATTERN-pane-text-injection.md` and `pattern/PATTERN-guarded-interrupt.md`.

Out:

- Creel stages 2 and 3 (`creel-notices`, `creel-agents`): orch strands stay withheld, and `lyx shuttle interrupt`, tender's pause and the orch watcher keep today's guarded interrupt.
- Resizing or re-laying-out a collapsed pane to deliver into it.

## Decisions

### D1 — #657: blank Claude's placeholder from a styled capture

Reed gains a styled pane capture, the screen with its SGR attributes, as a provider-blind read beside `CapturePane`.
Shuttle gains an optional engine capability that turns a styled capture into a plain one with the provider's placeholder cells blanked.
Wherever shuttle reads the input box of an engine with that capability (the idle probe with and without a run record, the pre-write guard, the landing check, and creel's interrupt reads before the key, after the settle and after the clear), it takes one styled capture and derives the plain view from it, so the running-turn and box readings of one decision come from the same screen.
Every other classifier keeps the plain capture.
`claudeengine` implements it: a box interior whose text is drawn wholly in the style Claude uses for its placeholder and prompt suggestion reads as empty.
Text in any other style, a box mixing typed text with placeholder-styled text, and a box matching the collapsed-paste placeholder shape are never blanked.
The style is pinned from a live capture of a fresh session, kept as a test fixture.
This widens what `PATTERN-pane-text-injection` calls an empty box for Claude; its background gains the placeholder bound beside the lone `%`.
The caret, the side bars and a lone `%` end-of-line mark are stripped before the style is checked, as `InputBoxText` and the end-of-line check strip them today, so placeholder-styled text beside a lone `%` reads as empty.
Bound: a box is blanked only when every non-blank cell of the box's text, once the caret, side bars and a lone `%` are stripped, carries the placeholder style, so an operator's typed text, which Claude draws unstyled, still reads as a draft; a Claude release that restyles the placeholder falls back to today's fail-closed reading (not idle), never to typing into a draft.

Rationale: the style is the one signal that tells the placeholder from a draft.

### D2 — #658: creel shows why a strand cannot take a delivery

The Delivery seam's idle answer carries a stall reason beside its idle flag: the pane is too short to show the input box, or the box holds a draft.
A running turn, a dialog or any other not-idle reading carries no reason.
`internal/creelcli` maps shuttle's `IdleProbe.TooShort` and `IdleProbe.Draft` onto it, and shuttle's pane-only probe for a strand with no run record fills `Draft` too.
The engine keeps the last reason per receiver with the time it was read, and journals a `stalled` record (receiver, reason) when the reason first appears or changes while something is pending; a delivery, or a reading with no reason, clears it without a record.
`log` prints `stalled` records.
A waiting `inject --wait` gets a progress line on its open connection whenever its receiver's reason changes; the progress line is a wire message the client keeps and writes to stderr as it arrives, never to stdout.
The verb still prints one JSON envelope on stdout, which carries the last reason and its time as fields when the wait ends expired or times out on the client side; an `expired` record carries the reason too.
The too-short reason names a way forward that is a real `lyx` command an agent's settings allow (`PATTERN-no-denied-recovery`).
The TTL keeps running while a stalled strand is live and unpaused, as `doc.go` states today.
Bound: one journal line and one progress line per change of reason per receiver, so an operator typing a draft adds at most one of each per typing episode while something is pending.

Rationale: naming the reason meets the issue's Expected and C13's "or creel names why"; the draft reason is included because #657 and #659 were both silent stalls behind a box holding text, and D3 falls back on it.

### D3 — #659: creel's interrupt clears the restored prompt

Shuttle gains an interrupt variant for creel, a new entry point of `PATTERN-guarded-interrupt`.
It resolves the strand's engine as `Runner.Interrupt` does, falling back to the live strand's provider for a strand with no run record, and reads the settle from that engine's `InterruptGuard`, not from `Runner.InterruptSettle`, which needs a run record.
It reads the box, sends the interrupt key only through `guardedInterrupt`, waits the settle, and reads the box again.
When the box was empty before the key and holds text after, it clears the box once through the engine's restored-prompt clear, behind the same three-fact guard reading the stray-character clear takes (session ready, no operator active within `operator_quiet_s`, box readable), and reads it again.
If the box still holds text after the clear, nothing more is sent: the inject waits, and D2 shows the draft as its stall reason.
A failure in the steps after the key (a capture, the guard reading or the clear) is logged and returned as a clear failure beside a sent interrupt, so creel journals `interrupted`, reports `sent:true` and still queues an `inject --interrupt` line.
`internal/creelcli`'s Delivery.Interrupt uses the variant, so both `lyx creel interrupt` and `inject --interrupt` get it; `Runner.Interrupt` and its other callers are unchanged.
While creel's interrupt runs for a receiver, the receiver's delivery worker neither probes nor sends nor records a stall, and the interrupt wakes it when it ends, so no queued inject or reminder is typed, or cleared, inside the settle window.
The restored-prompt clear is its own engine sequence, declared in `claudeengine` beside `ClearInputSequence`, which stays unchanged, so the stray-character and sent-text clears keep their one `C-u`.
It is sized from the post-settle read: one `C-u` per line of the box's text, with one `BSpace` between each pair to join the line above, so a multi-line prompt with the caret at its end is emptied; the plan confirms the caret position and the sequence from a live capture.
When the post-settle read finds the box empty, the worker stays suspended for one more settle and the box is read once more, and a prompt seen then is cleared the same way.
A prompt drawn after that second read is not cleared by D3: the worker resumes, and its pre-write guard reads either an empty box, so the inject is sent, or a draft, so the inject waits and D2 shows the draft reason.
A restore that lands between the pre-write guard's read and the paste-submit is submitted with the paste;
that window opens no earlier than two settles after the key and lasts only from the guard's read to the paste, and its worst outcome is the stopped turn's own prompt re-sent beside the inject, never an operator's draft.
This adds a second bounded clear to `PATTERN-pane-text-injection` beside the stray-character clear, and its background lists it among the explicit key deliveries.
Bound: the clear fires only on text that appeared in an empty box across lyx's own interrupt with no delivery of creel's in between, at most once per interrupt, and never without a passing guard reading; its sequence holds no Escape, `C-c` or Enter, only deletes box text, and is no longer than two keys per line the read showed; that text is the prompt of the turn just stopped, already in the transcript, so clearing it loses nothing.

Rationale: the restored prompt is a side effect of creel's own key, so creel's path owns undoing it.

### D4 — #660: a failed read write acknowledges nothing

The read verb's handler writes its envelope with a checked write and returns the write error, so `Client.Call` sends no acknowledgement, the messages stay unread and the verb exits non-zero.
The nothing-unread path checks its write the same way.
`output.Ok` keeps its signature; the check is in the creel read path.
A partial write followed by an error leaves the messages unread, so a consumer may hold a truncated envelope of messages it reads again later.

Rationale: the design already acks only on a nil handler return; the defect is the swallowed error.

### D5 — #661: the daemon's stderr goes to a per-hub log file

`hubgeom`'s creel Geometry declares a daemon log path in creel's state directory beside the journal, under the board worktree's never-tracked `.lyx`; neither the spawn nor the daemon verb builds the path itself.
The spawning client opens it for append, truncating it first when it exceeds a fixed cap, and points the detached daemon's stderr at it.
The daemon writes one start header line (pid and time) and raises its logger's stderr threshold to Warn, so its start error and its delivery and journal warnings reach the file.
Every line the daemon writes through its logger carries its pid, so two daemons started at once, whose lines interleave in the one file, stay apart.
When the client gives up connecting, its error names the file and quotes the lines carrying the pid of the process it spawned, plus pid-less lines (a runtime panic trace) that follow its own start header with no other start header after it, or says the file holds no line from that start.
When another start header follows its own, the error quotes only the pid-tagged lines and says the file also holds lines it cannot attribute.
A log file that cannot be opened does not stop the spawn: the daemon starts with null stderr, and a give-up error says the log could not be opened.
Bound: the file grows only by start headers, start errors and warnings, each warning logged once per change by `worker.report`, and the cap bounds it; a concurrent client's truncation past the cap can erase this start's lines, and the give-up error then says the file holds no line from that start;
a truncation that erases this start's header but not its later lines still finds those lines by pid.

Rationale: a file catches every start failure, panics included, without a second channel between client and daemon, and the daemon's delivery warnings are lost to null stdio the same way today.

### D6 — #662: TTL charged from each inject's posting

A probe charges each queued inject the time since the later of the worker's previous probe and that inject's posting, clamped at zero.
The same rule applies to the TTL counted at the worker's finish.
The posting time is the daemon Clock's time at the append; a carried or folded inject keeps its journaled consumed TTL, and the first probe after a start still charges nothing.

Rationale: the TTL is per inject, so its start is per inject.

### D7 — #663: read prints the returned mark

`read` prints `ended_unread_by` and `ended_unread_at` for a message the removed-holder sweep returned.

Rationale: the fold already holds both fields; only the rendering is missing.

### D8 — #664: output and help consistency

- `log` prints a `reminded` record with its message ids and its backoff step, and no `id` of 0.
- `inject --help`, and every other creel help text stating the privilege rule, says what `doc.go` says: EXTERNAL callers, programs and orch strands may inject, interrupt and hold on any strand, and any other strand on its own strand only.
- Every verb that prints a sender, the waiting ones included, prints the resolved sender.
- `release` first ends the caller's live hold on the name, so a hold on a strand removed since can still be released; a name that is neither a held receiver nor an existing one is refused with the names that exist, as `hold` is, and releasing an existing receiver the caller does not hold still answers `released:false`.

### D9 — Sandbox scenarios C1 to C13 beside S9

C1 to C13 go into `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` after S9, each with `**Covers:** creel`, a Goal, a Watch and the verdict the report gives, ids kept as `C1` to `C13`.
The suite's ref range, session-log template and notes name them.
The fixtures the audit used (bash strands, a fake idle pane that swallows input, a fake input box that ignores Enter, an orch-role fake) are described in the suite as setup steps.
A "Controlled creel exceptions" subsection names the non-black-box steps the scenarios need: listing the tmux and lyxtemp socket directories read-only (C1), planting or editing a journal day file while the daemon is stopped (C10), and a process detached from a pane (C12).
C12's disagreeing-strand-name check is operator-assisted when the agent's harness refuses a command naming the strand variable.
C13's verdict follows D1 and D2: a plain `--cmd claude` strand receives its reminder, and a 3-row strand's stall reason shows in `log` and `inject --wait`.

Rationale: the board entry names this file's S9 as the place, and one suite keeps the shuttle-plus-creel live setup in one session.

## Constraints

- `PATTERN-creel-store`: every keystroke stays behind the Delivery seam, and `internal/creelengine` imports neither reed nor shuttle; D2 and D3 change the seam's shape, not its route.
- `PATTERN-shuttle-provider-seam`: Claude's placeholder style and restored-prompt behaviour live in `internal/shuttleengine/claudeengine`; reed's styled capture interprets nothing.
- `PATTERN-guarded-interrupt`: D3's variant is a new guarded entry point whose key goes only through `guardedInterrupt`; its clear is the engine's input-box clear, never an interrupt key.
- `PATTERN-pane-text-injection`: changed by D1 (placeholder-only box reads empty) and D3 (restored-prompt clear), each with its bound; delivery still types only through reed's guarded paste-submit.
- `PATTERN-told-geometry` and `PATTERN-lyxdirs-single-declarer`: the D5 log path is declared in `hubgeom`'s creel Geometry and told.
- `PATTERN-durable-vs-ephemeral-state`: the D5 log is ephemeral and sits under `.lyx`.
- `PATTERN-tolerant-records`: the new `stalled` record and the added `expired` field are ignored by an older binary's fold.
- `PATTERN-no-denied-recovery`: the D2 too-short reason names only a command agents may run.
- `PATTERN-log-levels`: D2 and D5 log per change, never per probe.
- `PATTERN-sandbox-coverage`: each new scenario carries `**Covers:** creel`.

## Auto-mode assumptions

- D1: Claude draws its placeholder and its prompt suggestion in one distinct style that `tmux capture-pane -e` preserves, and draws typed text without it; the plan confirms both from a live capture before writing the fixture.
- D3: Claude restores the interrupted prompt into the box only after an interrupt early in a turn, and an Esc late in a turn leaves the box empty, so the clear is a no-op there.
- D3: Claude draws the restored prompt into the box within its `InterruptSettle`, with the caret at the prompt's end; the plan confirms both from a live capture.
- D9: the sandbox suite's agent can run bash fixtures in reed strands and stop the daemon by replacing its binary, as the audit's agent did.

## Open risks

- A Claude release that changes the placeholder style returns #657's symptom; the llm smoke test is what notices.
- If a Claude release moves the caret off the restored prompt's end or changes how `C-u` and `BSpace` edit a multi-line box, D3's sequence leaves text behind; D3 then sends nothing more and D2 reports a draft.
- The pre-write guard and landing check reading the blanked capture changes what a send sees as an empty box for every Claude run; shuttle's existing send tests must still pass on it.
- When `creel-agents` moves `lyx shuttle interrupt` behind creel, its callers gain D3's clear.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d3-multiline-clear-criterion-passes-single-cu (class design, severity MEDIUM, open in rounds 2)
  - d5-concurrent-starts-interleave-log (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `lyx gate test --tags llm ./internal/creelcli` passes with `TestSmokeCreelClaudeStrand` adding the strand with the bare claude command.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass, and the tmux tier passes for every package the change touches, `internal/creelcli`, `internal/shuttleengine` and `internal/reedengine` among them.
- A styled capture of a fresh Claude session's box with only the placeholder reads idle; the same box with typed text, with typed text followed by placeholder-styled text, and with the collapsed-paste placeholder reads not idle.
- A strand whose probe answers too short or draft gets one `stalled` record per change of reason, shown by `log`; `inject --wait` that times out on the client or expires prints one envelope carrying the reason and its time, with progress only on stderr; an `expired` record carries the stall reason; the too-short reason names a `lyx` command an agent's settings allow; the TTL still runs.
- After creel's interrupt, a box empty before the key and holding text after is cleared once; a box that held text before is left; an active operator holds the clear; a box still holding text after the clear sends nothing more; for a box read as n lines, the restored-prompt clear sends n `C-u` with one `BSpace` between each pair and nothing else; on a fake whose `C-u` deletes only the caret's line and whose `BSpace` at a line start joins the line above, a two-line and a three-line restored prompt are emptied, and a single `C-u` leaves text.
- No queued inject or reminder is typed into, or cleared from, a receiver during creel's interrupt and its settle.
- A clear that fails after a sent key leaves the interrupt journaled and reported as sent, and an `inject --interrupt` line still queued.
- The creel interrupt clears a restored prompt on a strand with no run record.
- `lyx creel read` into a writer that fails exits non-zero, with messages and with nothing unread, and any messages stay unread for the next read.
- A daemon that fails to start leaves the verb's error naming the daemon log file and quoting its error line from that start, never a line from another start.
- An inject posted while its receiver's worker is mid-backoff is charged no TTL from before its posting, and a daemon restart charges none for the time it was down.
- `read` of a returned message prints `ended_unread_by` and `ended_unread_at`.
- `log` prints reminded ids and step; every verb that prints a sender, waiting or not, prints the resolved one; every creel help text that states the privilege rule states it as `doc.go` does; `release` of an unknown name is refused naming the existing ones, and a hold on a removed strand can be released.
- C1 to C13 appear in the shuttle sandbox suite with `**Covers:** creel`, and the sandbox suite's own tests pass.

## Notes for the plan writer

- D1 touches every place shuttle captures for a box reading; routing them through one capture helper keeps the plain-versus-blanked choice in one spot.
- D2's seam change reaches `internal/creelengine`'s fake delivery in its tests and `internal/creelcli`'s `hubDelivery.Idle`.
- D2's too-short wording should be able to stand in for the orch watcher's `paneTooShortReason` when `creel-notices` drops the watcher's probe.
