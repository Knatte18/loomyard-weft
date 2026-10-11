# Decision record: shuttle-hook-first

## Goal

Shuttle reads a Claude session's state from its hook lines whenever they settle it, and reads the pane only for what no hook carries: startup gates, the input box and its drafts, a menu the operator opened, and operator activity.
Folded in: real hook payload fixtures as the first step, a bounded rule for a lone stray character in an idle input box (#554), and a tail-bounded read of fork transcripts.

## Scope

In: D1 to D9 below, with their docs in the same commit: `internal/shuttleengine/doc.go`, `internal/shuttleengine/claudeengine/doc.go`, and `pattern/PATTERN-pane-text-injection.md` (see Constraints).

Out:

- reed's session-name repair: stays pane-only.
- `Runner.SessionIdle` for a strand with no run record: stays pane-only.
- `requireReadyAgentPane` before `Runner.Interrupt` and before the send of an engine without `SessionCycler`: unchanged.
- The turn-start hold's pane idle release of an interrupted turn the transcript does not mark: unchanged.
- The startup-gate detection and its trust-dismiss sequence: unchanged in what they detect and when they send keys.
  This declines the board's step 3 rule that no gate key is sent once `SessionStart` lands.
- A `PostToolUse` heartbeat for batten's quiet notice, and `idle_prompt` as proof of an empty box: not built.
- The pane fallback for an unknown reading: kept, both for an engine without `SessionSignalParser` and for an unknown reading of Claude.

## Decisions

### D1. Capture the payload corpus first, in the task

The first card adds an `llm`-tier capture scenario in `internal/shuttlecli` beside `TestSmokeTurnsSessionHooks`, run by the implementer through `lyx gate test --tags llm`.
It drives real sessions through startup, a typed turn, a permission request, `/clear`, `/compact`, a resume, an `/exit` and a killed process.
For each hook line it records the hook name, the payload, and its order against pane captures: the trust or bypass gate on screen, and the first ready reading.
It also records Claude's session registry entry for the session when `SessionStart(startup)` lands, after `/exit` and after the kill.
The captured lines are trimmed into `claudeengine` corpus cases, and every payload-field rule below is tested against them.
`StopFailure` stays on its reconstructed fixture, since a real API error cannot be provoked on demand.
If the corpus shows that the `--settings` file's `SessionStart(startup)` line does not land on its own, with no turn typed, D4 cannot hold: the implementer reports that as a blocker in its batch report and does not build D4.
Every other Decision is built either way; D3's `startup` rule moves nothing when no `startup` line lands.
Under the blocker the Startup criterion and the #549 criterion drop, and every other criterion stands.
Rationale: the parsers' payload shapes are unverified against a real session, and rules keyed on `source`, `notification_type` or `reason` are only as good as those shapes.
A rule keyed on the hook name alone does not depend on the corpus.

### D2. Record `SessionStart` on every run, each source once

`buildSettings` installs a recording `SessionStart` hook on every run, for every source.
A run with a context-after-compaction command keeps that command, run after the recording command, on the `compact` source only.
Each `SessionStart` line is recorded exactly once whatever the source.
Rationale: the hook is the first signal a session gives, and today it exists only for runs with a compaction command.

### D3. A fold state for a started session with no turn yet

The fold gains the state `idle-fresh` with cause `session-start`.
A `SessionStart` signal sets the signal-derived state to `idle-fresh` when its source is `resume` or `clear`, or when its source is `startup` and the derived state is not a running turn.
A `SessionStart` signal with source `compact`, an empty source or any other source leaves the state unchanged.
`claudeengine` maps Claude's source strings to provider-neutral sources, so `shuttleengine` keys on no Claude string.
`judgeReadiness` reads `idle-fresh` like `idle-done`: ready, subject to the same too-short, no-box, unmatched-turn-start and draft checks.
The orch watcher's idle disagreement check and the wait shadow's pairing treat `idle-fresh` as an idle state.
The settle precedence is unchanged: a dead or unproven liveness and an unreadable events file still outrank `idle-fresh`.
Bound: a `compact` start never moves the state, so an automatic compaction inside a running turn keeps that turn busy;
a `startup` start never overrides a running turn, so a first turn whose `UserPromptSubmit` lands before the `SessionStart` line, as a launch-line first turn may, stays busy;
an unrecognised or missing source fails safe to today's behaviour.
Rationale: this removes the #549 path at its source, since the color send and the skill-load send after startup now read the hook state.
`resume` covers a relaunch after a crash mid-turn, and `clear` covers the `SessionEnd(clear)` that today leaves the fold unknown until the next turn.

### D4. Startup needs a `SessionStart` line and the pane's ready reading

For an engine whose launch records session starts, `checkLivenessTick` reads a run as started only once both hold: the events file holds a `SessionStart` line, and the pane reads `StartupReady`.
The rule lives in `checkLivenessTick`, so `awaitStartup` and an attached run's `Run.Wait` that is not yet started agree on it.
The engine declares that it records session starts through a provider-neutral capability or launch field; an engine that does not keeps today's pane-only startup.
The gate detection and dismissal run exactly as today until both hold, whatever the `SessionStart` line says.
A start that times out with a ready pane and no `SessionStart` line names that cause in its `abandonStartup` error, with the events file path; an unreadable events file reads as no line and names that.
Bound: a `SessionStart` line left by an earlier launch of the same run satisfies the hook half, and the pane half still gates it.
Rationale: a `SessionStart` line proves the process loaded lyx's settings.
If the corpus confirms that `--settings` hooks are held until the trust gate is accepted, it also proves the trust gate passed, so a gate caret the classifier misses can no longer read as ready.
The pane check stays because the docs do not tie `SessionStart` to the input box being drawn.

### D5. `PermissionRequest` reads asking at once; every `Notification` type is mapped

`buildSettings` installs a recording `PermissionRequest` hook on every run, and the parser maps its line to an ask by the hook name alone.
The transcript's interrupt marker releases an ask with cause `ask` to `idle-stalled` with cause `interrupt`, as it releases a running turn, so a dialog the operator escapes does not hold sends.
Bound: an ask holds every send until the next turn signal or an interrupt marker; a granted permission writes neither, so it reads asking for the rest of its turn.
The `Notification` mapping is a fixed table in `claudeengine`: `permission_prompt`, `elicitation_dialog` and `elicitation_url_dialog` are asks; `idle_prompt` is the idle notice; `agent_needs_input`, `elicitation_response` and every other type yield no signal.
Rationale: `PermissionRequest` fires when the decision is needed, where `Notification(permission_prompt)` follows about 6 s later and a keystroke defers it.
`agent_needs_input` is not an ask because the turn-end judgment already reads a waiting interactive run as asking, and an ask would override an `idle-done` reading of a finished run.

### D6. `/clear` and `/compact` are confirmed by their `SessionStart` lines

In `sendVerified`'s branch for a send that starts no turn, the clear-session and compact-session sends count as landed when a `SessionStart` signal with the matching provider-neutral source appears past the events offset taken before the send, the way `turnStartedSince` confirms text.
It waits no longer than the branch's existing landing window, and the box read stays as the fallback when no such line appears in it.
Rationale: neither command starts a turn, so no `UserPromptSubmit` confirms it, and the box read alone cannot tell a landed command from a lost one.

### D7. A proven dead process ends the run

`checkLivenessTick`, shared by `awaitStartup` and `Run.Wait`, reads the run's session liveness on each liveness tick and treats the session as dead when either holds:

- the newest signal is a `SessionEnd` whose reason ends the process, and the registry does not read the process alive;
- the registry proves the process gone: an entry for the session with a dead pid and no entry that leaves the answer open.

Registry death for a session id counts only once a liveness tick of the same wait has read that id alive; until then a dead registry reading leaves the answer unproven.
A process-ending `SessionEnd` counts only when its line lies past the events-file size this wait took at its first liveness tick, before that tick's registry read; an earlier `SessionEnd` leaves the answer unproven, whatever the registry later reads for its session id.
The liveness read uses the session id of the newest signal that names one, so a resumed session's own `SessionStart(resume)` line points the lookup at the live process.
A dead session then takes the same path as a dead pane: `OutcomeDone` when every output file exists, else the same `OutcomeDied` a dead pane gives, with no new transient marking.
The fold's settle precedence is unchanged: a process-ending `SessionEnd` reads dead in the fold even while the registry still reads the process alive, so sends are held for a session that announced its end, while the run ends only once the registry stops reading it alive.
Bound: an unproven liveness never ends a run, and a `SessionEnd` beside a registry entry whose pid is alive with a matching start time does not.
An earlier launch never fails a resumed launch, since a resumed launch shares the run's events file but its `SessionEnd` lies before this wait's first-tick size, and this wait never read the earlier process alive.
A `SessionEnd` written before this wait's first tick never counts, even when its process exits after that tick: such a run ends only on registry death or its deadline, as does a wait attached after the death.
Rationale: reed starts the pane as a login shell that outlives `claude`, so reed's pane liveness never sees a crashed Claude, and the run waits out its deadline today.

### D8. A lone stray character in an idle box is cleared once, then read as residue

`claudeengine` decides whether a box's text is a stray-character candidate: exactly one character that is neither a letter, a digit nor whitespace, beside the lone `%` rule it already owns.
When a send's guard reading finds a candidate in an otherwise idle box, and no operator was active within `operator_quiet_s`, shuttle sends the provider's clear-input sequence once, right after that reading with no wait or other key between them, waits the provider's redraw settle, and takes a second full guard reading: ready, no operator activity within `operator_quiet_s`, and the box read.
That post-clear reading is the write's guard reading.
This applies on both readings a send takes: the hook-state reading, and the pane-fallback reading of an unknown state, where the pane reads idle but for the candidate.
A not-ready session or operator activity holds the send as the guard does; an empty box proceeds; a box that still shows the same lone character reads as screen residue Claude never received, and the send proceeds as for the lone `%`; anything else holds as a draft.
Bound: the clear fires only for a candidate, only behind the full guard reading, and at most once per send attempt.
An operator loses the punctuation character they left for longer than `operator_quiet_s`, plus anything typed on that line in the latency between the guard reading and the clear.
A real character that survives the clear and the settle is submitted at the head of the message by the paste-submit's own Enter.
The post-paste confirm keeps its exact-text reading and takes no residue reading: a box holding the character beside the sent text, when that Enter did not take, fails the send with `ErrSentTextInDraft`, quoting the box and with no key, and records the pending send, as a merge with an operator's draft does today;
the box then holds later sends as a draft until the operator clears it.
Every hold for a draft quotes the box text it found, on the pane-fallback reading too, and batten's driver-busy notice to the orch carries that text; `lyx loom resume`'s refusal already carries it and keeps it.
Rationale: #554's `❯ +` blocked a resume until the operator attached; clearing keeps the delivered text exact in every case but the last bound, where treating the character as empty would always submit it.

### D9. Fork transcripts are read from the tail

`holdsPendingToolCall` reads a fork transcript backward from its end, finds the newest assistant message that holds `tool_use` blocks, and reports a pending call when any `tool_use` of that message has no `tool_result` after it.
A message is every assistant entry sharing one `message.id`, since Claude writes each parallel call as its own entry; the read continues back until it has passed every entry of that message.
It reads nothing older.
Rationale: an assistant message follows only after every result of the previous one, so older calls are answered; the cost is then bounded by the tail, not the transcript length, on every turn-end activity read of Merriam.

### Added after Discussion (parent, 2026-10-11): D7: SessionEnd counts only past the first-tick events-file size

Decision: A process-ending SessionEnd counts only when its line lies past the events-file size this wait took at its first liveness tick, with the size taken before that tick's registry read; the read-alive disjunct is dropped. Add a Death criterion: an earlier launch's process-ending SessionEnd as the newest signal, after this wait has read the resumed process alive, followed by an unproven registry reading, ends nothing.

Rationale: Round-3 Discussion review F1: the read-alive disjunct keys on the session id, so on a same-id resume an earlier launch's SessionEnd plus one transient unproven registry read ends the resumed run died, breaking D7's own bound; the disjunct adds nothing for D7's cases since a process read alive writes its SessionEnd after the first tick.

## Constraints

- `PATTERN-shuttle-provider-seam`: Claude's hook names, payload fields, source strings, notification types and the stray-character judgment live only in `claudeengine`; `shuttleengine` sees provider-neutral signal kinds, sources and box readings.
- `PATTERN-completion-signal`: the dead-session path of D7 is a new negative answer and consults `allOutputFilesExist` first; the enforcement test's audited sites are updated with it.
- `PATTERN-pane-text-injection`: D8 changes this entry's background in four places.
  The lead sentence's "never merges with, submits or wipes an operator's draft" gains D8's bounded exceptions, each held to D8's guard reading, `operator_quiet_s` and one clear per send attempt: the clear wipes a stray-character candidate, and the residue reading merges a real character that survives the clear and the settle with the sent text and lets the paste-submit's Enter submit it.
  The guard bullet's "One fresh reading takes three facts" gains D8's second reading: after a clear, the post-clear reading takes the same three facts, with the residue reading admitted for the box, and is the write's guard reading.
  The sentence on the lone `%` gains D8's residue reading: a lone stray-character candidate that persists after one clear and the redraw settle reads as empty, so a real character that survives both is merged into and submitted with the sent text, as an operator's `%` already can be.
  The residue reading is shuttle's, taken only on the post-clear reading, beside the provider's box reading rather than inside it; the `%` reading stays the provider's and applies at every reading.
  The list of explicit key deliveries outside the rule gains D8's clear into a box holding a stray-character candidate.
  The `PATTERN.md` line itself, "only reed's paste-submit, behind shuttle's pre-write guard or, for name repair, reed's activity check; no Enter of lyx's own into a box not holding exactly the sent text", is unchanged: the paste-submit's Enter belongs to reed's atomic paste-submit, which the `%` reading already lets merge with a box's character, and the post-paste confirm keeps its exact-text match, so no Enter of lyx's own goes into a box holding the character beside the sent text (D8).
- `PATTERN-daemon-wakeups`: no new wait; D4 adds one events-file read per startup probe and D7 one registry read per liveness tick, on the existing cadences.
- `PATTERN-test-speed` and `PATTERN-test-economy`: the capture scenario is `llm`-tier and run through `lyx gate test --tags llm`; the fold, parser, readiness, startup and transcript rules are tested at the lowest tier over the corpus cases and fakes, extending existing tables where they fit.
- An engine without `SessionSignalParser` behaves exactly as today on every path this task touches.

## Auto-mode assumptions

- The implementer can run the `llm`-tier capture through `lyx gate test --tags llm`, which the agents' raw-go deny allows.

## Open risks

- If the registry entry is not yet written when `SessionStart(startup)` lands, the fold reads unknown with liveness unproven and the color and skill-load sends take the pane fallback, as today; D8 covers a stray character on that path, and D1 records the timing.
- A granted permission mid-turn reads asking for the rest of that turn, by D5's bound, as `Notification(permission_prompt)` already does; batten or the orch watcher may show the run asking for that span.

## Acceptance criteria

- The capture scenario's corpus cases parse to the expected signals for `SessionStart` with each source, `PermissionRequest`, `UserPromptSubmit`, `Stop`, `Notification` and `SessionEnd`, and record the order of `SessionStart(startup)` against the trust gate and the first ready pane.
- Every run's settings record `SessionStart` for every source, `compact` included, once each; a run with a compaction command also runs that command on `compact` only.
- Fold: `SessionStart(startup)` alone reads `idle-fresh`; `SessionEnd(clear)` then `SessionStart(clear)` reads `idle-fresh`; a `resume` start after a turn start with no turn end reads `idle-fresh`; a `startup` start after an unmatched turn start stays busy; a `compact` start inside a running turn stays busy; an empty or unknown source leaves the state unchanged; a dead or unproven liveness outranks `idle-fresh`.
- Readiness: `idle-fresh` with an empty box is ready; with a draft, a too-short pane or no box it is held with that reason.
- The #549 case: a ready pane whose events file holds only `SessionStart(startup)` takes the color and skill-load sends from the hook state, without the pane fallback.
- Startup: a ready pane with no `SessionStart` line is not started and times out naming the missing line; an unreadable events file times out naming that; a `SessionStart` line with a gate on screen still dismisses the gate and waits for the ready pane; an earlier launch's `SessionStart` line with a pane not yet ready is not started; an attached not-yet-started run applies the same rule; an engine without session-start recording starts on the pane alone.
- `PermissionRequest` reads asking; an interrupt marker past that ask reads `idle-stalled` with cause `interrupt`; each `Notification` type maps as D5 states, and an unknown type yields no signal.
- `/clear` and `/compact` confirm on a matching `SessionStart` past the pre-send offset, and fall back to the box read when none arrives in the landing window.
- Death: a process-ending `SessionEnd` with the registry not alive ends the run died, `/exit` included; a session the wait read alive whose registry entry then reads a gone pid ends it died; with every output file present either ends it done; an unproven liveness and a `SessionEnd` beside a live registry entry end nothing, while the fold reads that session dead; a crashed launch's dead entry before a resumed launch reads alive ends nothing; an earlier launch's process-ending `SessionEnd` as the newest signal, with the resumed process not yet registered, ends nothing, and after this wait has read the resumed process alive, an unproven registry reading beside that line ends nothing too; a process the wait read alive after its `SessionStart` line that dies before the pane is ready fails the start as died.
- Stray character: a box holding only `+` with no operator activity is cleared and the send goes out with exact text, on the hook reading and on the pane-fallback reading; a lone `+` still on screen after the clear and settle is read as residue and the send goes out; operator activity in the settle holds the send at the post-clear reading; a residue send whose post-paste box holds `+` beside the sent text fails with `ErrSentTextInDraft`, sends no key and records the pending send; a lone `+` with operator activity, and a two-character box, hold with the text quoted; batten's driver-busy notice quotes the box text.
- Fork transcript: a parallel batch whose earlier call is pending and whose last call is answered reads pending; a fully answered newest message reads not pending whatever older messages hold; a transcript with no tool use reads not pending.

## Notes for the plan writer

- The capture scenario belongs beside `TestSmokeTurnsSessionHooks` in `internal/shuttlecli/smoke_turns_test.go`, whose doc says promoting a kept events file to a corpus case is a copy and a trim.
- `waitshadow.go`'s `loopAgreesWithState` and `internal/orchengine/watcher.go`'s idle disagreement check both enumerate idle state names.
