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
- Resume's reason text for a send the guard refuses.
- A new PATTERN entry for the guarded interrupt, in the same commit.

Out:

- Closing or detecting an open rewind picker or other dialog at resume.
- Holding or stopping Go producers and `lyx shed step` processes under a pause; they keep running by the design `lyx pause` shipped with, and agent spawns, shuttle run starts and network calls stay held at their entry points.
- Interrupting a turn that starts after the pause's interrupt step, such as an agent woken when its background task ends.
- Recovering the two driver panes already left in the picker on 2026-10-12; the operator closes them.

## Decisions

### D1 — The pause reads only a running turn as busy

`shuttleengine` gains one predicate on `SessionState` for a running turn: state `busy` with any cause but `background`.
`judgeReadiness` already applies this rule inline for sends and switches to the predicate, so the two share one state rule.
They still differ in the state they apply it to: send readiness first adjusts the folded state for an unmatched turn start and its hold, and the pause reads the raw folded state (Open risks).
The pause's grace wait, its interrupt step and resume's pre-send check use it: `pauseengine`'s `Pair.Busy` contract becomes "in a running turn", and `pausecli`'s `shuttlePair` fills it from the predicate.
A session reading `busy`/`background` therefore does not hold the grace and gets no Esc.
Rationale: such a session's turn has ended and its prompt is idle; an Esc cancels nothing there and does not stop the background shell.

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
- `refused` and `skipped`: unchanged; a failed guard capture lands in `refused`.

Rationale: `no_running_turn` keeps apart the sessions resume will type into, and makes a guard that withholds every Esc visible rather than hiding it as idle.

### D6 — Resume names a refused send by the guard's own reason

Resume keeps the guarded paste-submit and sends no Esc.
A send refused with `ErrSessionBusy` is reported with the readiness reason the guard named, such as "no input box is on screen; a menu or picker may be open", "a turn is running", a draft, or an operator-activity hold, without the pane tail, instead of the fixed "the input box holds a draft".
Rationale: the guarded send already refuses to type into a picker, so no line is lost into one; only the reason was wrong, and the guard's own reason tells a picker from a draft without detecting the picker.

### D7 — The guarded interrupt is a PATTERN entry

A new entry in `PATTERN.md`'s "Agents and prompts" group: sending an interrupt key to an agent pane goes only through shuttle's guarded interrupt, which sends only into a pane showing a running turn, and a retry waits the provider's settle time.
`pattern/PATTERN-pane-text-injection.md`, which lists `Runner.Interrupt` among the explicit key deliveries outside its rule, points at the new entry.
`Runner.Inject` stays the one raw key path that skips the ready-pane check, and the entry names it as exempt from the guard but forbidden the interrupt: Inject refuses an input list equal to the engine's interrupt sequence, sends nothing and returns an error naming `lyx shuttle interrupt`.
`creel`'s inject-with-interrupt therefore reaches its Esc through the guarded `Interrupt` only, as D3 says.
Rationale: every future interrupt caller, `creel` first, must follow the rule, and the repo puts a new cross-cutting invariant in PATTERN in the same commit.
Bound: the refusal is checked in shuttle against the engine's own sequence, so no Inject caller can carry a bare interrupt past the guard; an input list that holds Escape among other keys, such as a dialog dismissal, still goes through.

### Added after Discussion (parent, 2026-10-12): The rewind picker never appears; if it does, lyx treats it as an error

Decision: No lyx path may open Claude Code's rewind picker in a session it drives: every Esc lyx sends must be guaranteed not to reach an idle prompt or to double up. If lyx nevertheless sees the picker in a pane it drives (pause, resume, shuttle send or interrupt), that is an error, not a state to work around: lyx closes it with a single Esc, re-checks that the pane shows the input box again, and reports it loudly as a failure, never types into it and never selects an entry. Tests cover the picker being opened as a failure.

Rationale: Operator, 2026-10-12: the operator never uses rewind; it must never show up, and if lyx brings it up that is a bug. The approved design only refuses to type into an open picker, which leaves the session stuck.

## Constraints

- `PATTERN-shuttle-provider-seam`: the running-turn pane reading, the settle time and the Escape key stay in `internal/shuttleengine/claudeengine`; `shuttleengine`, `pauseengine` and `orchengine` see provider-neutral capabilities and import no provider package.
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
- A resume over a pane with no input box waits shuttle's send-ready timeout per session before it reports the refusal.

## Acceptance criteria

- A covered session reading `busy`/`background` does not hold the grace, gets no Esc, is not recorded in the entry, and appears under `background` with its outstanding task labels.
- A covered session in a running turn whose first Esc frees the pane gets exactly one Esc and appears under `interrupted`.
- A covered session whose state still reads a running turn after the first Esc, but whose pane shows no running turn after the settle, gets no second Esc and appears under `interrupted`.
- A second Esc is sent only after the settle time has passed and only when the pane still shows a running turn; after it the engine waits the settle again and classifies the session by the pane reading then, under `interrupted` when the pane shows no running turn and `still_busy` when it does.
- A session whose state reads a running turn but whose pane shows none at the first Esc gets no Esc, appears under `no_running_turn`, stays in the entry and gets the resume line on resume.
- A failed pane capture in the guard sends no key, and the pause lists the session under `refused`.
- `Runner.Interrupt` on a pane with no running turn sends no key and returns `ErrNoRunningTurn`; `lyx shuttle interrupt` reports it as a failure naming `lyx shuttle send`.
- `Runner.Inject` given an input list equal to the engine's interrupt sequence sends no key and returns an error naming `lyx shuttle interrupt`; an input list holding Escape among other keys still plays, and the existing Inject refusal tests that pass a lone `Escape` to reach another refusal switch to a non-interrupt input.
- A resume during the settle wait, in the pause engine or the orch watcher, ends the loop with no second Esc.
- The orch watcher sends its second Esc only on a tick past the settle and through the guard; a guarded refusal is treated as an idle orch and logged once per hub entry at Info naming the orch strand, with no Warn.
- A session recorded in an entry before this change that reads `busy`/`background` at resume gets a line attempt through the guarded send; a pane with the picker open yields the guard's reason, "no input box is on screen; a menu or picker may be open".
- A resume send refused with `ErrSessionBusy` is reported with the guard's own readiness reason.
- By hand: `lyx pause` against a driver idle at its prompt with a background `lyx shed step` sends no Esc and opens no picker, and against a driver mid-turn sends one Esc and leaves no picker.

## Notes for the plan writer

- The background state comes from `judgeTurnEnd` in `internal/shuttleengine/sessionstate.go`; the readiness rule D1 extracts is in `judgeReadiness` in `readiness.go`.
- `claudeengine` already matches the running-turn hint in `IdleSession` and `StrayInput`; the new capability reuses that needle and leaves `Startup` alone, which `run-window-bugs` plans to rework.
- `Run.Interrupt` has no production caller; its tests drive a fake pane, which must now show a running turn for them to expect a key.
- The orch watcher's loop is `handleHubEntry` in `internal/orchengine/pause.go`; its `Clock` has `Now()` only, which the cross-tick settle needs and no more.
- `creel` and `orch-split` will later move the orch watcher's delivery; the guard in shuttle is what they inherit.
