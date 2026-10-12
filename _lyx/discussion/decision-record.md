# Decision record — pause-esc-rewind

## Goal

A `lyx pause` sends Esc only into a pane that shows a running turn, and never sends two Escs close enough together to open Claude Code's rewind picker.
A session whose turn ended with background work still running is reported as such, not as busy.
The pause report's lists say plainly what each one holds and which sessions resume will type into.

## Scope

In:

- The pause sequence in `internal/pauseengine`: which sessions hold the grace, which get an Esc, the retry between the first and second Esc, and the report lists.
- The orch watcher's hub-pause interrupt in `internal/orchengine`: its retry after the first Esc and its handling of a guarded refusal.
- A pane-guarded interrupt in `internal/shuttleengine`'s `Runner.Interrupt` and `Run.Interrupt`, with the provider-specific pane reading and settle time under `internal/shuttleengine/claudeengine`.
  This changes `lyx shuttle interrupt`: on a session with no running turn it now fails instead of sending Esc.
- One provider-neutral "running turn" predicate on the session state in `shuttleengine`, shared by the pause and by shuttle's send readiness.
- Resume's reason text for a send the guard refuses, and the removal of its separate busy pre-check.
- Detecting an open rewind picker in shuttle's send and interrupt paths, closing it with one guarded Esc and failing the operation (D8).
- A new PATTERN entry for the guarded interrupt and the guarded picker close, in the same commit.

Out:

- Closing or detecting any dialog other than the rewind picker.
- Holding or stopping Go producers and `lyx shed step` processes under a pause; they keep running by the design `lyx pause` shipped with, and agent spawns, shuttle run starts and network calls stay held at their entry points.
- Interrupting a turn that starts after the pause's interrupt step, such as an agent woken when its background task ends.
- Recovering the two driver panes already left in the picker on 2026-10-12; the operator closes them.

## Decisions

### D1 — The pause reads only a running turn as busy

`shuttleengine` gains one predicate on `SessionState` for a running turn: state `busy` with any cause but `background`.
`judgeReadiness` already applies this rule inline for sends and switches to the predicate, so the two share one state rule.
They still differ in the state they apply it to: send readiness first adjusts the folded state for an unmatched turn start and its hold, and the pause reads the raw folded state (Open risks).
The pause's grace wait and its interrupt step use it: `pauseengine`'s `Pair.Busy` contract becomes "in a running turn", and `pausecli`'s `shuttlePair` fills it from the predicate.
A session reading `busy`/`background` therefore does not hold the grace and gets no Esc.
Resume drops its separate `Pair.Busy` pre-check and lets the guarded send's readiness decide, which applies the turn-start hold; a running turn at resume is reported by the guard's own reason (D6).
Rationale: such a session's turn has ended and its prompt is idle; an Esc cancels nothing there and does not stop the background shell.
Resume reads the raw folded state no longer because that state still reads a running turn for a session whose turn-start hold released, which `no_running_turn` promises the resume line.

### D2 — A background session is reported, not recorded

A covered session reading `busy`/`background` at the interrupt step goes into a new report list `background`, each item naming the strand and the labels of its outstanding background tasks from `SessionState.Outstanding`.
It is not recorded in the pause entry, so resume gives it no line.
Rationale: the operator sees what keeps running under the pause, which `go_producers` misses for a run whose driver session is live, and the task labels already sit in the session reading.
Bound: the list reports; it holds nothing and sends nothing.

### D3 — Every interrupt key is guarded by the pane

`Runner.Interrupt` and `Run.Interrupt`, after the existing ready-agent-pane check, capture the pane and send the interrupt key only when the capture shows a running turn.
When it shows none they send nothing and return a new sentinel `ErrNoRunningTurn`, whose message gives the way forward: nothing to interrupt; `lyx shuttle send` gives the agent a turn.
A capture that fails sends nothing and returns the capture error, so the guard fails closed.
The running-turn pane reading is a provider capability implemented in `claudeengine` from its existing running-turn needle; the fake engine in `internal/testkit/shuttlefake` implements it.
`ErrNoRunningTurn` means "nothing to stop": a caller that needs only the turn stopped, as the pause, the orch watcher and the planned `creel` inject-with-interrupt do, treats it as success; `lyx shuttle interrupt` reports it as a failure with that message.
The orch watcher logs a guarded refusal of its interrupt once per hub entry at Info, naming the orch strand, and never at Warn, so a guard that withholds every orch Esc stays visible though the pause report shows the orch only as delegated to the watcher.
Rationale: the pane is the only reading of what an Esc will do when it lands; the session state lags the transcript.
Bound: the guard can only withhold a key, never add one.
It lets through an Esc whose turn ends between the capture and the key, which costs one Esc on an idle prompt, and one Esc opens nothing.
It also reads a capture whose transcript quotes the running-turn hint as a running turn, since the needle matches anywhere in the capture, so it lets an Esc through onto that idle prompt.
lyx's own retries wait the settle time (D4), so two of their Escs land at least the settle time apart and open no picker.
Two operator-run `lyx shuttle interrupt` calls inside the settle time onto such a pane can open it; the next send or interrupt then closes it and fails (D8).

### D4 — A second Esc waits out a provider settle time and goes through the guard

The provider's interrupt settle time is a `claudeengine` constant of 2 seconds, longer than Claude Code's double-Esc window, exposed through the engine.
A caller that retries an interrupt waits at least the settle time after its first Esc before it reads again, and sends the retry through the guarded interrupt.
The maximum stays two Escs per session per pause.

- Pause engine: the settle wait goes through `Deps.Sleep`, and the settle time reaches the engine through its `Pair` seam, never by importing a provider package.
  The entry-live check and the claim stay before each Esc, so no Esc follows a resume.
  After the first Esc and the settle, the running-turn predicate on the state decides whether a second Esc is tried; a guarded second Esc refused with `ErrNoRunningTurn` ends the loop as interrupted.
  After a second Esc that went, the engine again waits the settle time through `Deps.Sleep`, then reads the pane through the guard's running-turn capability, reached through the `Pair` seam; that reading alone decides between `interrupted` and `still_busy`, since the state lags the transcript.
- Orch watcher: it never blocks a tick on the settle.
  After the first Esc it remembers, in memory per hub entry, when that Esc went, and a later tick whose `Now()` is past the settle re-probes and, if the orch still reads busy, sends the guarded second Esc.
  That tick runs the watcher's existing entry-live check first, so a resume during the settle ends the loop with no second Esc.
  `handleHubEntry` consults the remembered first-Esc time before `settledWithoutInterrupt`, whose entry-owns-the-orch check would otherwise settle the entry the first Esc's record made, and marks the entry handled only after the second Esc or the settled re-probe that finds the orch idle.
  The settle time reaches the watcher through its `Session` seam.
  Bound: a watcher restarted during the settle has lost the remembered time, finds the orch recorded in the entry, settles it and sends no second Esc.

Rationale: the live test's second Esc landed on a prompt the first had already freed; the settle and the guard together mean a second Esc goes only to a turn still on screen two seconds later.

### D5 — Report lists and what resume does with each

- `interrupted`: the turn stopped after an Esc: the state reads no running turn after the first Esc's settle, the guarded second Esc found no running turn on the pane, or the pane reading after the second Esc's settle shows none (D4).
- `no_running_turn`: the state read a running turn at the interrupt step, but the guarded first Esc found none on the pane.
  These sessions are recorded in the entry and get the resume line, whose text already tells a session with nothing cut to end its turn.
- `already_idle`: no running turn and no background work when the interrupt step reached it; not recorded, no line.
- `background`: the turn had ended with background work outstanding; not recorded, no line (D2).
- `still_busy`: the pane reading after the second Esc's settle still shows a running turn (D4).
- `refused` and `skipped`: unchanged; a failed guard capture and an open rewind picker (D8) land in `refused`, the picker with the guard's error as the reason.

Rationale: `no_running_turn` keeps apart the sessions resume will type into, and makes a guard that withholds every Esc visible rather than hiding it as idle.

### D6 — Resume names a refused send by the guard's own reason

Resume keeps the guarded paste-submit, and its only Esc is D8's picker close inside that send.
A send refused with `ErrSessionBusy` is reported with the readiness reason the guard named, such as "no input box is on screen; a menu or picker may be open", "a turn is running", a draft, or an operator-activity hold, without the pane tail, instead of the fixed "the input box holds a draft".
A send that finds the rewind picker open fails with D8's error, reported with that error's message, and the resume line is withheld.
Rationale: the guarded send already refuses to type into a dialog, so no line is lost into one; the reason was wrong, and the guard's own reason tells a draft from a turn or a hold, while D8 names the rewind picker by detecting it.

### D7 — The guarded interrupt is a PATTERN entry

A new entry in `PATTERN.md`'s "Agents and prompts" group: sending an interrupt key to an agent pane goes only through two guarded paths in shuttle.
The guarded interrupt sends only into a pane showing a running turn, and a retry waits the provider's settle time.
The guarded picker close (D8) is the one named exception: it sends exactly one interrupt key, only into a pane the provider reads as the rewind picker, and never a second.
`pattern/PATTERN-pane-text-injection.md`, which lists `Runner.Interrupt` among the explicit key deliveries outside its rule, points at the new entry.
`Runner.Inject` stays the one raw key path that skips the ready-pane check, and the entry names it as exempt from the guard but forbidden the interrupt: Inject refuses any input list containing a key of the engine's interrupt sequence, whatever its `SettleMS` and whatever keys surround it, sends nothing and returns an error naming `lyx shuttle interrupt`.
`creel`'s inject-with-interrupt therefore reaches its Esc through the guarded `Interrupt` only, as D3 says.
Rationale: every future interrupt caller, `creel` first, must follow the rule, and the repo puts a new cross-cutting invariant in PATTERN in the same commit.
Bound: the refusal is checked in shuttle against the keys of the engine's own sequence, so Inject can deliver every key but the provider's interrupt key, and no Inject caller can carry one past the guard.

### D8 — An open rewind picker is closed once and fails the operation

Added after Discussion by the parent, 2026-10-12, and authoritative over the record's earlier text.
No lyx path may open Claude Code's rewind picker in a session it drives: D1 to D4 keep every Esc off an idle prompt and apart by the settle time.
If lyx nevertheless finds the picker in a pane it drives, that is an error, not a state to work around.

- Detection: a provider capability in `claudeengine` reads a pane capture as the rewind picker, from a needle on the picker's own heading pinned by a captured-pane fixture.
  `shuttleengine` reaches it provider-neutrally, as it reaches D3's running-turn reading; the fake engine in `internal/testkit/shuttlefake` implements it.
- Close path: a guarded picker close in `shuttleengine`, separate from the guarded interrupt, which refuses a pane with no running turn.
  It captures the pane and, only when the capability reads the capture as the picker, sends exactly one key of the engine's interrupt sequence.
  It then waits the provider's settle time, through a sleep tests inject, and re-captures; the operation fails with a new sentinel `ErrRewindPickerOpen` either way.
  Its message says the picker was open and was closed, and when the re-capture shows no input box it says the picker or another dialog is still on screen.
- Bound: at most one Esc per detection, sent only when the capture shows the picker, never an entry selected, never a key typed after it in the same operation.
  A re-check that still shows no input box sends nothing more; the failure is reported and the pane is left to the operator.
  The picker close cannot double up with a guarded interrupt in the same call, since the operation ends at the close.
  A picker the operator closes between the capture and the key costs one Esc on an idle prompt, which opens nothing.
- Where it runs: first thing after the ready-agent-pane check in `Runner.Interrupt`, `Run.Interrupt` and shuttle's send path, before the running-turn reading or the send readiness.
  - Pause: the guarded first or second Esc returns `ErrRewindPickerOpen`, and the session lands in `refused` with its message (D5).
  - Orch watcher: logs the error at Warn naming the orch strand, sends no further Esc for that hub entry and marks it handled.
  - Resume: the send fails, the line is withheld even when the close succeeded, and the session is reported with the error's message (D6).
  - `lyx shuttle send` and `lyx shuttle interrupt`: exit with a failure carrying the error's message, nothing typed and no interrupt sent.

Rationale: Operator, 2026-10-12: the operator never uses rewind; it must never show up, and if lyx brings it up that is a bug.
Refusing to type into an open picker alone leaves the session stuck; one Esc frees it, and failing the operation keeps the bug loud.
Withholding the resume line after a close leaves the operator to read the pane before the session gets a turn.

## Constraints

- `PATTERN-shuttle-provider-seam`: the running-turn pane reading, the rewind-picker reading, the settle time and the Escape key stay in `internal/shuttleengine/claudeengine`; `shuttleengine`, `pauseengine` and `orchengine` see provider-neutral capabilities and import no provider package.
- `PATTERN-orch-pane-single-writer`: the orch watcher stays the only Go writer into the orch pane; its hub-pause interrupt keeps that exemption and gains only the guard and the settle.
- `PATTERN-pane-text-injection`: resume's line stays on reed's guarded paste-submit; this entry's background changes as D7 says.
- `PATTERN-daemon-wakeups`: the watcher's settle rides its existing ticks and adds no wake.
- `PATTERN-tolerant-records`: the pause entry's shape does not change; the new lists live in the report only.
- `PATTERN-test-speed` and `PATTERN-test-economy`: engine-level tests run on fakes with an injected clock and sleep; extend `pause_test.go`, the `orchengine` pause tests and the shuttle interrupt tests rather than adding parallel suites.
- Docs in the same commit: `pauseengine/doc.go`, the orch watcher's pause doc, the `lyx shuttle interrupt` help and PATTERN (D7).

## Auto-mode assumptions

- Claude Code's double-Esc window is under 2 seconds; the settle time is set at 2 seconds on that assumption.
- One Esc ends a running Claude Code turn in the normal case, so the second Esc stays only as a guarded, settled retry.
- `lyx shuttle interrupt` failing on an idle session is acceptable observable behaviour; an Esc on an idle prompt never did anything useful.
- The orch watcher is in scope for its immediate re-probe after the first Esc; its idle probe already reads an orch idle with background work as ready, so it never sent that orch an Esc.

## Open risks

- A session in `background` wakes when its task ends, and that turn runs under the pause uninterrupted; its network calls and spawns wait at their entry points, and a suspend that cuts its API call is covered by the existing API-error continue.
- The running-turn needle reads Claude Code's spinner hint; a release that rewords it makes the guard withhold every Esc, and the report then lists those sessions under `no_running_turn`; the orch's case shows in the watcher's Info line for the refusal, not in the pause report.
- The pause and send readiness share the state rule, not the whole reading.
  A session whose folded state reads idle while its pane shows a running turn, an unmatched turn start, gets no Esc and lands in `already_idle`.
  A `busy`/`turn` state whose turn-start hold released reads running to the pause and reaches the guard, which withholds the Esc when the pane shows no running turn, so the session lands in `no_running_turn`.
  At resume no raw-state check stands before the guarded send, whose readiness applies the hold and lets the line through (D1).
- The rewind-picker needle reads Claude Code's picker heading; a release that rewords it leaves the picker undetected, and the send and interrupt paths fall back to the guards' existing refusals, so nothing is typed into it.
- A resume over a pane with no input box waits shuttle's send-ready timeout per session before it reports the refusal.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - inject-unbounded-interrupt-key-path (class design, severity MEDIUM, open in rounds 1, 2)
  - parent-picker-decision-contradicts-record (class consistency, severity BLOCKING, open in rounds 2)
  - picker-close-esc-no-mechanism-or-bound (class design, severity BLOCKING, open in rounds 2)
  - resume-busy-precheck-refuses-no-running-turn (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A covered session reading `busy`/`background` does not hold the grace, gets no Esc, is not recorded in the entry, and appears under `background` with its outstanding task labels.
- A covered session in a running turn whose first Esc frees the pane gets exactly one Esc and appears under `interrupted`.
- A covered session whose state still reads a running turn after the first Esc, but whose pane shows no running turn after the settle, gets no second Esc and appears under `interrupted`.
- A second Esc is sent only after the settle time has passed and only when the pane still shows a running turn; after it the engine waits the settle again and classifies the session by the pane reading then, under `interrupted` when the pane shows no running turn and `still_busy` when it does.
- A session whose state reads a running turn but whose pane shows none at the first Esc gets no Esc, appears under `no_running_turn`, stays in the entry and gets the resume line on resume, including when its raw state still reads `busy`/`turn` with the turn-start hold released.
- A failed pane capture in the guard sends no key, and the pause lists the session under `refused`.
- `Runner.Interrupt` on a pane with no running turn sends no key and returns `ErrNoRunningTurn`; `lyx shuttle interrupt` reports it as a failure naming `lyx shuttle send`.
- `Runner.Inject` given an input list containing a key of the engine's interrupt sequence sends no key and returns an error naming `lyx shuttle interrupt`, with rows for a lone Escape, an Escape with a `SettleMS`, two Escapes, and Escape among other keys; an input list with no interrupt key still plays, and the existing Inject refusal tests that pass a lone `Escape` to reach another refusal switch to a non-interrupt input.
- A resume during the settle wait, in the pause engine or the orch watcher, ends the loop with no second Esc.
- The orch watcher sends its second Esc only on a tick past the settle and through the guard; a guarded refusal is treated as an idle orch and logged once per hub entry at Info naming the orch strand, with no Warn.
- A session recorded in an entry before this change that reads `busy`/`background` at resume gets a line attempt through the guarded send; a pane with the rewind picker open gets one Esc, is re-checked for the input box, is reported as a failure with `ErrRewindPickerOpen`'s message, and gets no line typed.
- The guarded picker close, in `Runner.Interrupt`, `Run.Interrupt` and shuttle's send path: a capture read as the rewind picker gets exactly one Esc and no further key, the pane is re-captured after the settle time, and the call returns `ErrRewindPickerOpen`; a capture not read as the picker gets no Esc from it.
- When the re-capture after the close still shows no input box, no further key is sent and the error says a dialog is still on screen.
- The pause lists a session whose guarded Esc found the picker under `refused` with the error's message; the orch watcher logs it at Warn naming the orch strand and sends no further Esc for that entry; `lyx shuttle send` and `lyx shuttle interrupt` fail with the error's message.
- A resume send refused with `ErrSessionBusy` is reported with the guard's own readiness reason.
- By hand: `lyx pause` against a driver idle at its prompt with a background `lyx shed step` sends no Esc and opens no picker, and against a driver mid-turn sends one Esc and leaves no picker.

## Notes for the plan writer

- The background state comes from `judgeTurnEnd` in `internal/shuttleengine/sessionstate.go`; the readiness rule D1 extracts is in `judgeReadiness` in `readiness.go`.
- `claudeengine` already matches the running-turn hint in `IdleSession` and `StrayInput`; the new capability reuses that needle and leaves `Startup` alone, which `run-window-bugs` plans to rework.
- No captured pane of the rewind picker sits in the repo yet; the picker capability's fixture comes from a capture of a real picker, such as those left open on 2026-10-12 if the operator has not yet closed them.
- `Run.Interrupt` has no production caller; its tests drive a fake pane, which must now show a running turn for them to expect a key.
- The orch watcher's loop is `handleHubEntry` in `internal/orchengine/pause.go`; its `Clock` has `Now()` only, which the cross-tick settle needs and no more.
- `creel` and `orch-split` will later move the orch watcher's delivery; the guard in shuttle is what they inherit.
