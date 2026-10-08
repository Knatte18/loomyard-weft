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

An engine without the idle reading keeps today's `requireReadyAgentPane` guard; an engine without session signals skips the second condition.
A session that is not idle within the window fails the send with an error wrapping a new sentinel `ErrSessionBusy`, naming the reading that failed and carrying the pane's last lines (D5).

Rationale: the skill-load send ran straight after the startup probe passed, and the gate's sends after an attach ran on a boundary read from replayed events; both typed into a session that was not ready.
The hook's `UserPromptSubmit` is a fact Claude Code reports, so a turn start after the last turn end proves a running turn regardless of what the replayed `Stop` lines suggest.

Bound: this is the first behaviour consumer of the hook signals that `shuttle-state` added in shadow mode.
It reads only the raw `turn_start` signal, never the fold's derived state, and can only delay a send or fail it as busy; it never ends, classifies or finalizes a run.
The wait is capped by `send_ready_timeout_s`.

### D2. The typed text settles before Enter

Decision: for an engine that reads its input box (`InputBoxReader`), the send choreography separates typing from submitting.
After the text is typed, shuttle reads the input box until two consecutive reads, one `submit_settle_ms` apart, show the same content, then sends Enter.
A collapsed paste placeholder counts as settled content.
An engine without the capability keeps today's fixed choreography (`ComposeSend` with its fixed settle).
The settle wait counts against the submit window of D3.

Rationale: under machine load a long text is not consumed within the fixed 300 ms settle, and an Enter landing inside the typing burst is read as part of a paste and swallowed.

### D3. Submission is confirmed within a time window with backoff

Decision: after Enter, the input box is re-read at a growing interval, starting at `submit_redraw_settle_ms` and doubling up to 5 s, until the submission is confirmed or `submit_confirm_timeout_s` (new key, default 30) has passed.
Each read that shows the box still holding the sent text sends one more Enter; a read that shows an empty box, a running turn or a draft lacking the needle sends none, exactly as today's guard.
A submission is confirmed when the box no longer holds the sent text, or, for an engine with session signals, when a `turn_start` signal appears past the events offset read before typing.
`sendExtraEnters` is retired; the count of Enters is bounded by the window and the backoff.
A submission not confirmed within the window fails with an error wrapping a new sentinel `ErrSubmissionNotLanded`, carrying the pane's last lines (D5).

Rationale: a fixed two extra Enters over about one second cannot outlast a loaded machine; a bounded time window with backoff can, without spamming Enters.
The `turn_start` signal is positive proof of submission and only ever ends the wait early; the box reading stays the only evidence that a submission did not land.

Bound: the extra Enters still go out only into a box holding the sent text, so the guard that keeps an Enter off an empty box, a running turn or a foreign draft is unchanged; the window caps how long and therefore how many.

### D4. A start-time send that does not land is a not-started run

Decision: when the skill-load send or the prompt delivery in start fails with `ErrSessionBusy` or `ErrSubmissionNotLanded`, start tears the run down through the existing not-ready teardown (`abandonStartup`): the record is finalized to a terminal outcome, the strand removed, the last pane capture saved, and the error wraps `ErrNotStarted` beside the send's own sentinel.
Every other start failure past `AddStrand` (such as the prompt-offset persist) takes the same teardown, so no start error leaves a live strand behind a record reading `running`.
`shedengine.TransientAgentStart`'s doc comment widens to "an agent session that never became ready or never took its first input"; the class set and `shedtransient`'s mapping stay as they are, since `ErrNotStarted` already maps to it.

Rationale: an agent that never received its prompt has done no work, so re-running the step unchanged can succeed, which is the transient class's own test.
Reusing `ErrNotStarted` routes the failure through paths that already handle it: burler's `NotStarted` half, its attempt-2 retry, and the agent-start transient mark at the shed boundary.

Bound: no new transient class and no new marking site (PATTERN-transient-stop).
The widened class still excludes an agent that took its prompt and then died or timed out.

### D5. Send failures carry the pane's last lines

Decision: `ErrSessionBusy` and `ErrSubmissionNotLanded` errors end with the last 15 non-blank lines of the final pane capture, so the error text in the shed status file and the loom halt note shows the cause (busy agent, modal prompt, unconsumed paste).
A failed capture says so in place of the lines.

Rationale: #442 asks for the cause to be visible in the halt note, and the error text is what reaches it.

### D6. Mid-run sends retry instead of ending the gate

Decision: in the gated wait loop, the boundary is confirmed idle in fact (D1's signal condition) before the gate is evaluated for a send; a `turn_start` after the boundary's turn end clears the boundary and the loop keeps polling, with no send and no failure.
A re-prompt send that fails with `ErrSessionBusy` or `ErrSubmissionNotLanded` leaves the entry at the boundary with its failure count unchanged and is tried again on a later boundary, the way a failed pending send already is; any other re-prompt send error still ends the loop as today.

Rationale: `AttachGated` replays events from the prompt offset, so its first tick reads the writer as idle even mid-turn and could type a notice or a reject re-prompt into a running turn.
A re-prompt that did not land is an infrastructure miss, not a spent attempt.

Bound: the retry spends no gate budget, so it is bounded by the run deadline, which still reaches `classifyDeadlineExpiry` and the gate's final evaluation.

### D7. Shuttle owns stopping a run's strand, and records the stop

Decision: shuttle gains one stop verb for a run's strand, reachable from a run handle and by strand guid for a caller holding only the guid.
It writes the run's record to a new record-only terminal outcome `stopped` first, then removes the strand if reed reports it live.
`stopped` is recognised by `isTerminalOutcome` and never returned as a `Result.Outcome`.
Burler's half stop, webster's leftover-strand reclaim and attach's superseded-strand removal call it in place of their own `RemoveStrand` calls; burler's and webster's own removers go.
A record that is already terminal keeps its outcome; a concurrent `Wait` on the stopped run that finalizes afterwards may overwrite `stopped` with its own terminal outcome, and either answer is terminal.
A guid with no run record is stopped as today, by strand removal alone.

Rationale: burler stopped a half by removing its strand and left its record reading `running`; the next probe saw an untracked strand behind a young running record and refused, which is the duplicate-agent guard firing on a run lyx itself ended.
Shuttle owns the run directory, so only shuttle writes the record.
Writing the record before the removal means a crash between the two leaves a terminal record over a live strand, which attach already reads as respawn-eligible and removes as superseded; the reverse order would recreate the bug.

### D8. A new PATTERN entry pins the stop path

Decision: add `PATTERN-shuttle-stop` under "Agents and prompts": "Stopping the strand of a shuttle run from Go: only through shuttle's stop verb, which records the run terminal before the strand is removed; no other package removes a shuttle run's strand." It is review-enforced; a scan test is added only if the plan finds one cheap to write against the call sites.

Rationale: the bug came from a second stop path that did not know about the run record; a written invariant keeps a third from appearing.

### D9. Config keys and docs

Decision: `send_ready_timeout_s` (default 60) and `submit_confirm_timeout_s` (default 30) join shuttle's template with the same comment style as their neighbours; 0 or negative is a config error for both.
`submit_settle_ms` and `submit_redraw_settle_ms` keep their keys, now the settle-read interval (D2) and the first backoff step (D3); their template comments are updated to say so.
`shuttleengine`'s `doc.go` describes the send's idle wait, settle, confirm window and stop verb; `claudeengine`'s and `burlerengine`'s `doc.go` follow their changes.

Rationale: the windows are machine-load dependent, so an operator on a slow host needs to widen them without a rebuild.

## Constraints

- `PATTERN-shuttle-provider-seam`: the idle reading, the input-box reading and the split choreography stay in `claudeengine`; `shuttleengine` reaches them only through capabilities.
- `PATTERN-transient-stop`: no new class, no new marking site; the widened definition is stated at the declaration in `shedengine`.
- `PATTERN-completion-signal`: attach's file-contract-first checks keep their precedence over the new `stopped` outcome; a negative "did this finish" answer still consults `allOutputFilesExist` first.
- `PATTERN-wait-mark-display-only`: the send waits are not tied to `@lyx_wait` decisions; if a wait mark is set while a send waits, it is set through `SetWaitMark` and read by no Go decision.
- `PATTERN-orch-pane-single-writer`: unchanged; the orch watcher keeps its own idle gate and simply inherits the safer send.
- `PATTERN-spawn-observability`: the stop verb logs the strand removal at Warn, as the removers it replaces did.
- `PATTERN-test-speed` and `PATTERN-test-economy`: send and stop behaviour is pinned in the untagged tier through the injected sleep and clock seams and the existing fakes; existing send tests are extended rather than duplicated.
- `PATTERN-comment-line-breaks` for every changed Go comment.

## Auto-mode assumptions

- No operator answered questions; every pick below is an auto-pick recorded in the support log's question ledger.
- Default windows: 60 s for the idle wait, 30 s for the submit window, 5 s backoff cap, 15 pane lines in an error.
- `stopped` as a record-only outcome rather than reusing `died`.
- Webster's reclaim moves onto the stop verb although it never hit the attach refusal, so one stop path exists.

## Open risks

- The root cause of #442 is not reproduced live; the design covers the three named candidates (unconsumed paste, session still starting, readiness probe passing too early), and D5 makes the next occurrence diagnosable from its error text.
- A session whose idle reading never passes (a greyed suggestion, a pane too short to draw its box) now fails a send as busy after the window instead of typing; the template's suggestion switch-off and the existing `PaneTooShort` reading are what keep that rare.
- Waiting up to `send_ready_timeout_s` inside `Runner.Send` lengthens `lyx shuttle send` into a busy agent from instant to the window before it fails; that is the intended trade for never typing into a running turn.
- D1's signal condition depends on `UserPromptSubmit` hook lines reaching the events file; a missing hook line makes the condition vacuous and falls back to the pane reading alone, never to a false busy.

## Acceptance criteria

- A send into a pane that shows a running turn or a non-empty draft does not type until the pane reads idle, and fails with `ErrSessionBusy` carrying the pane's last lines when the window passes.
- With session signals, a `turn_start` after the newest turn end holds the send even when the pane reads idle.
- Typed text whose box content still changes between reads gets no Enter until two reads agree.
- A box that still holds the sent text after the first Enter gets further Enters at a growing interval, and the send succeeds once the box clears within the window.
- A `turn_start` signal past the pre-send offset confirms a submission even while the box reading is ambiguous.
- No extra Enter is sent into an empty box, a running turn or a draft lacking the needle (existing guard, still pinned).
- A submission still pending at the end of the window fails with `ErrSubmissionNotLanded` carrying the pane's last lines.
- A skill-load or prompt-delivery send that does not land during start returns an error matching both `ErrNotStarted` and the send's sentinel, with the run's record terminal and its strand removed.
- A burler round whose second half's start does not land ends as that half's `NotStarted` died outcome, its first half stopped and recorded `stopped`, and the burler adapter's attempt-2 failure carries the agent-start transient class.
- An attach probe over a record reading `stopped` whose strand is gone answers respawn-eligible at any directory age, so the re-step that halted in #444 proceeds.
- A record reading `running` with an untracked strand younger than the age window still refuses as today.
- The stop verb writes the record before removing the strand; a strand-removal failure leaves the record terminal and returns the error.
- After an `AttachGated` over a writer mid-turn, no gate send is typed until a turn end follows the writer's newest `turn_start`.
- A gate re-prompt that does not land spends no attempt and is re-sent at a later boundary.
- Burler, webster and attach stop strands only through the stop verb (review check against `PATTERN-shuttle-stop`).
- The new config keys load from the template, and 0 or negative values are config errors.

## Notes for the plan writer

- The incident chain: `burlerengine`'s round start (`StartGated` of the fixer) failed in `loadSkillTurn`'s send, `stopHalf` removed the reviewer's strand through `reedStrandRemover`, the adapter's attempt 2 returned a hard error, and the re-step's probe of the reviewer half hit `leftoverThenAgeVerdict`'s `verdictError` over the reviewer's record still reading `running`.
- `loadSkillsThenPrompt` returns `run.identity(), err` on a send error today, leaving the strand live; that is the spot D4 routes through `abandonStartup`.
- `handleGatedBoundary` in `wait.go` holds both gate sends; the pending-send branch already shows the stay-pending pattern D6 extends to the re-prompt.
- `claudeengine/signals.go` already parses `UserPromptSubmit` into `SessionSignalTurnStart`; D1 and D3 need only an offset-bounded read of those signals, not the fold.
- `confirmSubmitted`'s existing table test in `run_test.go` is the natural place to extend for D2 and D3.
