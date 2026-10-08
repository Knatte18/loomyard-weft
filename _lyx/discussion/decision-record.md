# Decision record — shuttle-send

## Goal

A text shuttle sends into an agent pane lands, and when it cannot, the run retries instead of halting.
A shuttle run that lyx itself stopped is recorded as stopped, so a later attach probe never halts on it.
Covers issues #442, #443 and #444, and the "no send into a running turn after a restart" case moved here from `parent-review-hardening`.

## Scope

In:

- `internal/shuttleengine`'s verified send (`sendVerified`, `confirmSubmitted`, the readiness guard) and every path that reaches it: the skill-load turn and prompt delivery in start, `Run.Send`, `Runner.Send`, and the gate's pending and re-prompt sends in the wait loop.
- The provider side of that send in `internal/shuttleengine/claudeengine`: the send choreography and the idle reading of the pane.
- Start-time failures past `AddStrand`: none may leave a live strand behind a record reading `running`.
- A shuttle verb that stops a run's strand and records the stop, and its use by burler's half stop (`internal/burlerengine`), webster's leftover-strand reclaim (`internal/websterengine`) and attach's superseded-strand removal.
- Attach's classification of a record lyx stopped.
- The doc comment of `shedengine.TransientAgentStart`, widened to a session that never took its first input.
- New shuttle config keys with their template lines, `doc.go` updates for `shuttleengine`, `claudeengine` and `burlerengine`, and one new `PATTERN.md` entry.

Out:

- The unknown-run refusal in attach (`verdictError`: untracked or binding-cleared, younger than the age window) stays a hard refusal; only runs lyx stopped stop reaching it.
- `lyx reed remove` run by the operator stays a raw strand removal.
- Orch sessions (`internal/orchcli`'s strand removal), which have no shuttle run record.
- Moving any other consumer onto the session-state fold (batten, orch watcher, creel), which `shuttle-state` sequences as their own tasks.
- `Runner.Inject`, which deliberately types into a busy pane and keeps doing so.

## Decisions

### D1. Send waits for an idle session before typing

Decision: every verified send first waits until the session is idle in fact, polled with backoff up to `send_ready_timeout_s` (new key, default 60).
Idle in fact means both:

- the pane shows the provider idle: the engine's existing idle reading (`IdleSession`: input box present and empty, no running-turn hint) passes, on top of today's startup-ready classification;
- when the engine reads session signals (`SessionSignalParser`), no `turn_start` signal follows the newest turn end in the run's events file.
  A turn end is a `turn_end` or an `api_error_turn_end` signal.

An engine without the idle reading keeps today's `requireReadyAgentPane` guard; an engine without session signals skips the second condition.
A `turn_start` with no later turn end stops holding the send when the engine's `SessionProber.TurnStartInterrupt` reports that turn interrupted, or once the pane has read idle on every poll for 10 s; the override logs a Warn naming the unmatched signal.
An engine with session signals but no `SessionProber` has only the 10 s path.
A session that is not idle within the window fails the send with an error wrapping a new sentinel `ErrSessionBusy`, naming the reading that failed and carrying the pane's last lines (D5).
`ErrSessionBusy` is returned before any key is typed.

Rationale: the skill-load send ran straight after the startup probe passed, and the gate's sends after an attach ran on a boundary read from replayed events; both typed into a session that was not ready.
The hook's `UserPromptSubmit` is a fact Claude Code reports, so a turn start after the last turn end proves a running turn regardless of what the replayed `Stop` lines suggest.

Bound: this is the first behaviour consumer of the hook signals that `shuttle-state` added in shadow mode.
It reads only the raw `turn_start` signal, never the fold's derived state, and can only delay a send or fail it as busy; it never ends, classifies or finalizes a run.
The wait is capped by `send_ready_timeout_s`, and an unmatched `turn_start` (an interrupted turn, a missing `Stop` line) holds a send against an idle pane for at most the 10 s override.

### D2. The typed text settles before Enter

Decision: for an engine that reads its input box (`InputBoxReader`), the send choreography separates typing from submitting.
After the text is typed, shuttle reads the input box until two consecutive reads, one `submit_settle_ms` apart, show the same content, then sends Enter.
A collapsed paste placeholder counts as settled content.
An engine without the capability keeps today's fixed choreography (`ComposeSend` with its fixed settle).
Between typing and the settle reads, the existing appearance check (`sendVerified`'s poll for the text in the pane, `sendVerifyAttempts` polls `sendVerifyInterval` apart) still runs, and its replay still re-types a text that never appeared; both run inside the submit window of D3, so a replay is cut short when the window closes.
A text that has not appeared when the window closes or the replays run out fails the send with `ErrSubmissionNotLanded`, since its cause, a swallowed input, is the same miss.
The settle reads run inside that window too, which starts when typing begins; a box that has not settled when the window closes gets no Enter, and the send fails with `ErrSubmissionNotLanded`.

Rationale: under machine load a long text is not consumed within the fixed 300 ms settle, and an Enter landing inside the typing burst is read as part of a paste and swallowed.

### D3. Submission is confirmed within a time window with backoff

Decision: the submit window, `submit_confirm_timeout_s` (new key, default 30), opens when typing begins and covers D2's settle and this confirmation.
After Enter, the input box is re-read at an interval that doubles from `submit_redraw_settle_ms` up to 5 s, until the submission is confirmed or the window has closed.
An interval that would end past the window's close is cut to end at it, but the read after an Enter never comes sooner than `submit_redraw_settle_ms` after it.
A read taken while the window is open that shows the box still holding the sent text sends one more Enter; a read that shows an empty box, a running turn or a draft lacking the needle sends none, exactly as today's guard.
A read taken at or after the window's close sends no Enter and ends the loop.
Every Enter is therefore followed by one confirm read, so a send never fails without reading the box after its last Enter; that read is the window's only overshoot, one `submit_redraw_settle_ms` sleep and one capture.
A submission is confirmed when the box no longer holds the sent text, or, for an engine with session signals, when a `turn_start` signal appears past the events offset read before typing.
`sendExtraEnters` is retired; the count of Enters is bounded by the window and the backoff.
A submission not confirmed within the window fails with an error wrapping a new sentinel `ErrSubmissionNotLanded`, carrying the pane's last lines (D5).
Before returning `ErrSubmissionNotLanded`, the send clears its own text out of the box, so the box D1 requires empty is empty again for the next send.
The clear goes through a new engine-owned key choreography beside `ComposeSend`, sent only when a box reading taken just before shows it holding this send's text: copies of the sent text (a doubled replay), the last of which may be a prefix of at least the needle's length (an unsettled box), or a collapsed paste placeholder.
Only an engine with both the idle reading and the clear choreography clears: D1's idle reading proved its box empty before typing, so such content is this send's own.
An engine without the idle reading gets no clear, and the text stays in its box.
A box holding anything else, or reading empty, gets no clear key.
The choreography never uses a key that interrupts a running turn, so a turn that started between the reading and the clear is not cut short.
The box is read once more after the clear; a box still not empty is named in the error, and the next send fails as busy against it as today.

Rationale: a fixed two extra Enters over about one second cannot outlast a loaded machine; a bounded time window with backoff can, without spamming Enters.
The `turn_start` signal is positive proof of submission and only ever ends the wait early; the box reading stays the only evidence that a submission did not land.

Bound: the extra Enters still go out only into a box holding the sent text, so the guard that keeps an Enter off an empty box, a running turn or a foreign draft is unchanged; the window caps how long and therefore how many.

### D4. A start-time send that does not land is a not-started run

Decision: when the skill-load send or the prompt delivery in start fails with `ErrSessionBusy` or `ErrSubmissionNotLanded`, start tears the run down through the existing not-ready teardown (`abandonStartup`): the record is finalized `died`, the strand removed, the last pane capture saved, and the error wraps `ErrNotStarted` beside the send's own sentinel.
Every other start failure past `AddStrand` (such as the prompt-offset persist) takes the same teardown, so no start error leaves a live strand behind a record reading `running`.
`shedengine.TransientAgentStart`'s doc comment widens to "an agent session that never became ready or never took its first input"; the class set and `shedtransient`'s mapping stay as they are, since `ErrNotStarted` already maps to it.

Rationale: an agent that never received its prompt has done no work, so re-running the step unchanged can succeed, which is the transient class's own test.
Reusing `ErrNotStarted` routes the failure through paths that already handle it: burler's `NotStarted` half, its attempt-2 retry, and the agent-start transient mark at the shed boundary.

Bound: no new transient class and no new marking site (PATTERN-transient-stop).
The widened class still excludes an agent that took its prompt and then died or timed out.
`ErrSessionBusy` types nothing, so the agent holds no prompt to work on.
A prompt that landed late is caught twice before the teardown.
The box reading D3 takes before its clear shows an empty box, so the send succeeds instead of failing.
After `ErrSubmissionNotLanded`, start reads the session signals once more, and a `turn_start` past the pre-send offset makes the start succeed with the run handle instead; the box is not re-read, since the send's own clear has emptied it.
The residual is a prompt consumed between that signal read and the strand removal; its agent is killed within that gap, and if it wrote an output file the re-step's `Spec.validate` refuses on that file instead of re-running.

### D5. Send failures carry the pane's last lines

Decision: `ErrSessionBusy` and `ErrSubmissionNotLanded` errors end with the last 15 non-blank lines of the final pane capture, so the error text in the shed status file and the loom halt note shows the cause (busy agent, modal prompt, unconsumed paste).
A failed capture says so in place of the lines.

Rationale: #442 asks for the cause to be visible in the halt note, and the error text is what reaches it.

### D6. Mid-run sends retry instead of ending the gate

Decision: in the gated wait loop, the boundary is confirmed idle in fact (D1's signal condition, with its interrupt and 10 s idle-pane override) before the gate is evaluated for a send; a `turn_start` after the boundary's turn end clears the boundary and the loop keeps polling, with no send and no failure.
A boundary cleared by a `turn_start` that has no later turn end is restored on the tick where D1's override releases it: `SessionProber.TurnStartInterrupt` reports the turn interrupted, or the pane has read idle on every poll for 10 s.
While a boundary is cleared that way, the loop captures the pane every tick for D1's idle reading, which times the 10 s.
The restoring tick evaluates the gate through `handleGatedBoundary` as a new turn end does, whether or not an entry is pending; the unsent-re-prompt mark is not restored, since that evaluation re-derives any re-prompt from the gate's recorded state.
A re-prompt send that fails with `ErrSessionBusy` or `ErrSubmissionNotLanded` leaves the entry's failure count unchanged and marks the boundary as holding an unsent re-prompt; the loop's per-tick branch that today re-enters `handleGatedBoundary` for a pending entry at the boundary re-enters it for that mark too, so the gate is re-evaluated and the re-prompt re-sent on later ticks without a new writer event.
A writer turn that starts meanwhile clears the boundary and the mark, and its next turn end is evaluated afresh.
Any other re-prompt send error still ends the loop as today.
Inside the wait loop, a send takes one budget, the time left until the run deadline at the send's start: the idle wait ends by the deadline, and the submit window, which holds the appearance check, its replay, the settle and the confirmation (D2, D3), closes by it too.
A send therefore overshoots the deadline by at most its tails past it: the idle wait's last pane capture, D3's post-Enter confirm read, the clear with its key choreography and two box readings, and D5's final capture.

Rationale: `AttachGated` replays events from the prompt offset, so its first tick reads the writer as idle even mid-turn and could type a notice or a reject re-prompt into a running turn.
A re-prompt that did not land is an infrastructure miss, not a spent attempt.

Bound: the retry spends no gate budget; it is bounded by the run deadline, which still reaches `classifyDeadlineExpiry` and the gate's final evaluation.
A boundary cleared by a `turn_start` is progress, not a loop: either the writer is in a turn whose end becomes the next boundary, or D1's override restores the cleared boundary within 10 s of idle pane.

### D7. Shuttle owns stopping a run's strand, and records the stop

Decision: shuttle gains one stop verb for a run's strand, reachable from a run handle and by strand guid for a caller holding only the guid.
It writes the run's record to a new record-only terminal outcome `stopped` first, then removes the strand if reed reports it live.
A record write that fails returns the error and removes nothing.
`stopped` is recognised by `isTerminalOutcome` and never returned as a `Result.Outcome`.
Burler's half stop, webster's leftover-strand reclaim and attach's superseded-strand removal call it in place of their own `RemoveStrand` calls; burler's and webster's own removers go.
Burler stops a half through the handle form: its `Handle` seam gains the stop method, which `*shuttleengine.Run` implements, and `stopHalf` calls it in place of the guid remover, so a half stopped during `join` while its own `Wait` runs is marked.
Webster's reclaim and attach's removal hold no in-process `Wait` on the run, so they use the guid form.
A record that is already terminal keeps its outcome.
A record reading `running` whose output files all exist keeps `running` and only loses its strand: the stop verb consults `allOutputFilesExist` first, so attach's file-contract-first branch still harvests the run as finished.
`stopped` therefore means "lyx ended this run before its output files existed".
Stopped through a run handle, the verb also marks the handle, so that handle's later `Wait` finalize persists `stopped` unless its outcome is `done` or every output file exists (the choosing function in Constraints); a `done` reached through the file contract still wins.
The stop verb and the run's own `Wait` goroutine share `run.state` and its `run.json`: `Wait` saves `run.state` mid-run (the held-turn notice offset, `Started`) and in finalize.
`Run` therefore gains a record mutex guarding the mark, `run.state.Outcome` and every save of `run.state`; the stop verb holds it while it sets the mark and `run.state.Outcome` and saves, and every `Wait` save holds it, so a later mid-run save carries `stopped` instead of restoring `running`.
The mark is set before the strand is removed, so the `Wait` that sees the strand go already reads it.
The guid form reads `run.json` from disk, sets the outcome and writes it back; it has no in-process `Wait` to race.
A `Wait` in another process, holding no such mark, may overwrite `stopped` with its own terminal outcome, and either answer is terminal.
A guid with no run record is stopped as today, by strand removal alone.

Rationale: burler stopped a half by removing its strand and left its record reading `running`; the next probe saw an untracked strand behind a young running record and refused, which is the duplicate-agent guard firing on a run lyx itself ended.
Shuttle owns the run directory, so only shuttle writes the record.
Writing the record before the removal means a crash between the two leaves a terminal record over a live strand, which attach already reads as respawn-eligible and removes as superseded; the reverse order would recreate the bug.

### D8. A new PATTERN entry pins the stop path

Decision: add `PATTERN-shuttle-stop` under "Agents and prompts": "Stopping the strand of a shuttle run from Go outside `internal/shuttleengine`: only through shuttle's stop verb, which settles the run's record before the strand is removed; no other package removes a shuttle run's strand." Shuttle's own teardowns inside its run lifecycle (finalize's cleanup, `abandonStartup`) write the record themselves and stay as they are. It is review-enforced; a scan test is added only if the plan finds one cheap to write against the call sites.

Rationale: the bug came from a second stop path that did not know about the run record; a written invariant keeps a third from appearing.

### D9. Config keys and docs

Decision: `send_ready_timeout_s` (default 60) and `submit_confirm_timeout_s` (default 30) join shuttle's template with the same comment style as their neighbours; 0 or negative is a config error for both.
`submit_settle_ms` and `submit_redraw_settle_ms` keep their keys, now the settle-read interval (D2) and the first backoff step (D3); their template comments are updated to say so.
`shuttleengine`'s `doc.go` describes the send's idle wait, settle, confirm window and stop verb; `claudeengine`'s and `burlerengine`'s `doc.go` follow their changes.

Rationale: the windows are machine-load dependent, so an operator on a slow host needs to widen them without a rebuild.

### Added after Discussion (parent, 2026-10-08): D7 record write path

Decision: For a run held in-process, the stop verb records stopped by mutating run.state under the new mutex, and every saveRunState call on run.state from the Run (finalize, hold.go's NotifiedOffset save, wait.go's Started save) takes the same mutex; the mutex is a record-write lock, not a lock on one field. The guid form (no Run in process) stays a disk read-modify-write of run.json and has no in-process Wait to race. Finalize takes the stored outcome from the one function the PATTERN-completion-signal constraint names, so runOutcomeStopped appears at one return and both writes go through it; done still takes precedence when allOutputFilesExist holds.

Rationale: Round-3 F1 and F3: the stop verb's write shares run.state and run.json with Wait's own saves, and a second stopped write outside the named return would escape the tripwire.

### Added after Discussion (parent, 2026-10-08): D3 confirm loop end and overshoot

Decision: A box read taken after the submit window has closed sends no Enter and ends the loop. The sleep before the read an Enter triggers is submit_redraw_settle_ms; the doubling interval (from submit_redraw_settle_ms up to 5 s) spaces the Enters. D6 states the overshoot past the window as the union of the tails: the last confirm read, the clear and its readings, D5's capture, and the last poll of D1's idle wait.

Rationale: Round-3 F2: the loop had no stop rule after the window, and the stated overshoot bound did not hold.

### Added after Discussion (parent, 2026-10-08): Text never appeared is a landing failure

Decision: The appearance check's failure (the sent text never appeared in the pane, including when the window closes during it) wraps ErrSubmissionNotLanded, so D4 and D6 retry it like the other landing failures. D3's clear also matches a box holding the sent text repeated, the case a replay that typed twice leaves.

Rationale: Round-3 F4: the board's goal is that a send that does not land retries instead of halting; the TUI swallowing the input is the same class.

### Added after Discussion (parent, 2026-10-08): Clear premise, D6 restore, attach wording

Decision: D3's clear runs only for an engine that supplies both the idle reading and the clear choreography; any prefix of the sent text counts, and the Open risk names the paste-placeholder case. When D1's override releases a boundary cleared by an unmatched turn_start, the gate is evaluated on that tick even with nothing pending, the re-prompt is re-derived rather than a mark restored, and the wait loop reads the pane every tick while a boundary is cleared. The Constraint reads: isTerminalOutcome recognising stopped is the only change to attach's classification, and removeSupersededStrands moves to the stop verb. D1's acceptance criterion uses D1's own wording, idle on every poll for 10 s.

Rationale: Round-3 F5-F8: small specification gaps with one obvious resolution each.

### Added after Discussion (parent, 2026-10-08): PATTERN-shuttle-stop names the loom driver exception

Decision: The PATTERN-shuttle-stop entry names one exception beside shuttle's own teardowns: loomcli's removal of the loom driver's strand (driverPaneProbe.RemoveDriverStrand, reached from lyx loom start) stays a raw strand removal and is out of this task's scope. The entry reads: Stopping the strand of a shuttle run from Go outside internal/shuttleengine: only through shuttle's stop verb, which settles the run's record before the strand is removed; no other package removes a shuttle run's strand, except loomcli's removal of the loom driver's own strand.

Rationale: Plan-Write found that the loom driver is a shuttle run whose strand loomcli removes raw. No attach probe or Wait reads the driver's record, so the bug class of #444 does not reach it, and moving it is outside the decided scope; without the named exception the entry would be false the day it lands.

### Added after Discussion (parent, 2026-10-08): The segment color goes through the send path after readiness

Decision: After the merge with main (reed-view, f25395e49), three loomcli tmux-tier tests fail in publish_verify: TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver (the provider never became ready; the pane shows '/color blue' left in the input box), TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps and TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand (loom start timed out after 30s; 'could not resume the parked driver ... the strand's pane shows no input-ready provider'). This task owns the fix: every path that types reed-view's '/color' into a session (Start's playInputs of ColorSequence and Runner.TypeColor for a caller that colors itself, such as the loom driver) plays only after the provider is ready and goes through this task's send discipline (idle wait, settle, landing confirmed), so a color command never sits unsent in the input box and readiness never reads a box the color typing occupies. The color stays display-only: a color send that fails is logged and never fails the launch. Verify with the full go test ./... including the tmux-tier tests.

Rationale: reed-view landed a raw typed color while this task made sends and readiness strict; the two meet only on the merged branch, and the fix sits inside this task's own goal (a Send lands or retries).

### Added after Discussion (parent, 2026-10-08): Burler: the merged raw /color typing is a blocking finding

Decision: The Webster-Burler reviewer raises as a BLOCKING finding that run.go's playInputs(ColorSequence) after awaitStartup and Runner.TypeColor (session.go) still type /color raw, even though that code arrived through the merge of main and lies outside the task's diff range. The fix sends the segment color through shuttle-send's send path after readiness (idle wait, settled box, confirmed landing); a failed color send is logged and never fails the launch. The fixer runs go test -tags tmux ./internal/loomcli/... ./internal/shuttleengine/... and gets it green before finishing; it must pass TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver, TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps and TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand.

Rationale: publish_verify runs the tmux tier, which the Burler verify gate does not; the raw /color left text in the input box, so the stricter readiness never saw the provider ready and Publish blocked twice.

### Added after Discussion (parent, 2026-10-08): Orch strand removal is a second PATTERN-shuttle-stop exception

Decision: PATTERN-shuttle-stop names two exceptions: loomcli's removal of the loom driver's own strand and orchcli's removal of the orch strand. The Out list's premise that the orch strand has no shuttle run record is wrong: orchcli starts the orch session with runner.Start, so it has one; orch stays out of scope.

Rationale: No attach probe or Wait reads the orch run record, the same reasoning as the driver exception; moving orchcli's removals onto the stop verb would widen the task, and leaving the entry unchanged would make the invariant false on landing.

## Constraints

- `PATTERN-shuttle-provider-seam`: the idle reading, the input-box reading and the split choreography stay in `claudeengine`; `shuttleengine` reaches them only through capabilities.
- `PATTERN-transient-stop`: no new class, no new marking site; the widened definition is stated at the declaration in `shedengine`.
- `PATTERN-completion-signal`: writing `stopped` is a new negative "did this finish" answer, so the stop verb consults `allOutputFilesExist` first (D7).
  The record value is a constant `runOutcomeStopped`, chosen in one function that returns the outcome a stop leaves: the outcome it is handed when that is `done` or every output file exists, else `runOutcomeStopped`.
  The stop verb calls it on a `running` record, and a marked handle's finalize calls it on the outcome it classified, so the value has that one return site and that function's `allOutputFilesExist` call guards both writes.
  In `completionsignal_enforcement_test.go`, in the same commit: `runOutcomeStopped` joins `negativeVerdictMarkers`, the stop verb's file joins the scanned files, that function's return joins the audited return sites and its `allOutputFilesExist` call the audited call sites.
  `PATTERN-completion-signal`'s background file and `wait.go`'s file doc comment add `stopped` to their list of negative answers.
  Attach's file-contract-first branch is unchanged and a `stopped` record never reaches it; `isTerminalOutcome` recognising `stopped` is the only change to attach's classification, and its doc comment widens from the values finalize wrote to every terminal value a record holds.
  Attach's other change is its superseded-strand removal moving onto the stop verb (D7).
- `PATTERN-wait-mark-display-only`: the send waits are not tied to `@lyx_wait` decisions; if a wait mark is set while a send waits, it is set through `SetWaitMark` and read by no Go decision.
- `PATTERN-orch-pane-single-writer`: unchanged; the orch watcher keeps its own idle gate and simply inherits the safer send.
- `PATTERN-spawn-observability`: the stop verb logs the strand removal at Warn, as the removers it replaces did.
- `PATTERN-test-speed` and `PATTERN-test-economy`: send and stop behaviour is pinned in the untagged tier through the injected sleep and clock seams and the existing fakes; existing send tests are extended rather than duplicated.
- `PATTERN-comment-line-breaks` for every changed Go comment.

## Auto-mode assumptions

- No operator answered questions; every pick below is an auto-pick recorded in the support log's question ledger.
- `stopped` as a record-only outcome rather than reusing `died`, and a stopped run whose output files exist left `running` for attach to harvest.
- A fixed 10 s idle-pane override for an unmatched `turn_start`.
- A send that did not land clears its own text from the box (D3) rather than leaving it for the next send.
- Webster's reclaim moves onto the stop verb although it never hit the attach refusal, so one stop path exists.

## Open risks

- The root cause of #442 is not reproduced live; the design covers the three named candidates (unconsumed paste, session still starting, readiness probe passing too early), and D5 makes the next occurrence diagnosable from its error text.
- A session whose idle reading never passes (a greyed suggestion, a pane too short to draw its box) now fails a send as busy after the window instead of typing; the template's suggestion switch-off and the existing `PaneTooShort` reading are what keep that rare.
- Waiting up to `send_ready_timeout_s` inside `Runner.Send` lengthens `lyx shuttle send` into a busy agent from instant to the window before it fails; that is the intended trade for never typing into a running turn.
- D1's signal condition depends on hook lines reaching the events file.
  A missing `UserPromptSubmit` line makes the condition vacuous and falls back to the pane reading alone.
  A missing `Stop` line leaves an unmatched `turn_start` that delays a send into an idle pane, or the restoring of a gate boundary (D6), by up to D1's 10 s override.
- D3's clear removes whatever the box holds that matches this send's text; an operator who types or pastes into the pane while the send's window is open loses that input when it starts like the send's text or shows as a collapsed paste placeholder.
- D4 widens `TransientAgentStart` on an auto-pick; whether a session that never took its first input deserves its own class instead awaits the operator (support log Q8).

## Acceptance criteria

- A send into a pane that shows a running turn or a non-empty draft does not type until the pane reads idle, and fails with `ErrSessionBusy` carrying the pane's last lines when the window passes.
- With session signals, a `turn_start` after the newest turn end holds the send even when the pane reads idle, until `TurnStartInterrupt` reports it interrupted or the pane has read idle on every poll for 10 s; an `api_error_turn_end` counts as a turn end.
- `Runner.Send` into a busy pane waits for idle as every other verified send does.
- Typed text whose box content still changes between reads gets no Enter until two reads agree, and a box that never settles within the submit window fails with `ErrSubmissionNotLanded` and gets no Enter.
- A box that still holds the sent text after the first Enter gets further Enters at a growing interval, and the send succeeds once the box clears within the window; an Enter sent as the window closes is still followed by one confirm read, which sends no further Enter.
- A `turn_start` signal past the pre-send offset confirms a submission even while the box reading is ambiguous.
- No extra Enter is sent into an empty box, a running turn or a draft lacking the needle (existing guard, still pinned).
- A submission still pending at the end of the window fails with `ErrSubmissionNotLanded` carrying the pane's last 15 non-blank lines; a failed final capture says so in their place.
- A skill-load or prompt-delivery send that does not land during start returns an error matching both `ErrNotStarted` and the send's sentinel, with the run's record `died` and its strand removed.
- A prompt-delivery send whose box reads empty at the reading before the clear succeeds, and one that fails with `ErrSubmissionNotLanded` but whose `turn_start` has appeared by the signal read before teardown starts the run instead.
- A sent text that never appears in the pane fails with `ErrSubmissionNotLanded`.
- A send that fails with `ErrSubmissionNotLanded` leaves the box empty when it held copies of this send's text, a needle-long or longer prefix of it or a paste placeholder, sends no clear key into a box holding anything else or nothing, or on an engine without the idle reading, and names a box the clear did not empty in its error.
- A prompt-offset persist failure during start takes the same teardown: record `died`, strand removed, error wrapping `ErrNotStarted`.
- A burler round whose second half's start does not land ends as that half's `NotStarted` died outcome, its first half stopped and recorded `stopped`, and the burler adapter's attempt-2 failure carries the agent-start transient class.
- A burler half stopped during `join` because the other half ended first reads `stopped` after its own `Wait` returns.
- An attach probe over a record reading `stopped` whose strand is gone answers respawn-eligible at any directory age, so the re-step that halted in #444 proceeds.
- A record reading `running` with an untracked strand younger than the age window still refuses as today.
- The stop verb writes the record before removing the strand; a strand-removal failure leaves the record terminal and returns the error, and a record-write failure returns the error with the strand untouched.
- The stop verb keeps an already-terminal record's outcome, keeps `running` on a record whose output files all exist, and stops a guid with no run record by strand removal alone.
- A run stopped through its handle reads `stopped` after that handle's `Wait` finalizes with any outcome but `done`.
- After an `AttachGated` over a writer mid-turn, no gate send is typed until a turn end follows the writer's newest `turn_start`, or D1's override releases that `turn_start` (interrupted, or 10 s of idle pane), which restores the boundary.
- A gate re-prompt that does not land spends no attempt and is re-sent on a later tick with no new writer event, into the box its failed send cleared; a writer turn starting meanwhile clears the pending re-send.
- A send inside the wait loop that starts with less time left than its windows ends its idle wait and its submit window by the run deadline.
- Burler, webster and attach stop strands only through the stop verb (review check against `PATTERN-shuttle-stop`).
- The new config keys load from the template, and 0 or a negative value for either key alone is a config error naming that key.

## Notes for the plan writer

- `loadSkillsThenPrompt` returns `run.identity(), err` on a send error today, leaving the strand live; that is the spot D4 routes through `abandonStartup`.
- `handleGatedBoundary` in `wait.go` holds both gate sends; the wait loop's `run.gatePending && run.gateAtBoundary` branch is the per-tick re-entry D6 extends to an unsent re-prompt.
- `claudeengine/signals.go` already parses `UserPromptSubmit` into `SessionSignalTurnStart`; D1 and D3 need only an offset-bounded read of those signals, not the fold.
- `confirmSubmitted`'s existing table test in `run_test.go` is the natural place to extend for D2 and D3.
