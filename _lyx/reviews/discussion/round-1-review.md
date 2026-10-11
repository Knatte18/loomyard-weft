---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: D7's staleness test compares a registry entry's process start with the newest SessionStart line, but every process starts before its own SessionStart line, so the rule either voids all registry evidence or fails to protect a resumed launch
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints, PATTERN-pane-text-injection entry
    summary: D8's residue path submits a surviving real character merged with lyx's text, but the Constraints entry amends only the background's wipe rule and declares the PATTERN.md line, no Enter into a box not holding exactly the sent text, unchanged
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 and Open risks
    summary: An ask from PermissionRequest is never released by the interrupt marker, so a permission dialog the operator escapes leaves the run asking and every send held, a regression the lingering-ask Open risk does not cover
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D7 against the fold's settle precedence
    summary: D7 lets a live registry entry veto a process-ending SessionEnd in checkLivenessTick, but the record leaves the fold reading such a SessionEnd as dead, so readiness, the wait shadow and the orch watcher read dead while the run is not ended
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D1, D3 and Constraints
    summary: D1 says D3's startup rule cannot hold if SessionStart(startup) does not precede the first typed turn, while D3's bound and the Constraints line say D3 holds for either hook order
  - id: F6
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D1
    summary: D1's corpus blocker says the implementer builds neither D3's startup rule nor D4, but leaves open whether D3's other sources, D2, D6 and the #549 Acceptance criterion still stand, and the criteria are stated unconditionally
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D8 bound
    summary: D8's bound admits the loss of anything typed between the guard reading and the clear, but states no limit on that window, so the wipe exception to never wipes an operator's draft is unbounded in what it can destroy
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, Death and Startup bullets
    summary: The criterion that a dead process before startup is ready fails the start as died conflicts with D7's rule that only a SessionEnd signals death before any SessionStart line, and D4's earlier-launch line bound has no criterion
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Question ledger Q2
    summary: Q2 still says a SessionStart line proves trust passed either way and that D4 escalates through Open risks, which the record now makes conditional on the corpus and moved into D1
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Auto-mode assumptions, Open risks, Constraints
    summary: Auto-mode assumptions two and three, the third Open risk and the shuttle-color-verified-send Constraints line restate D1, D4 and D3 and carry no requirement of their own
---

### [BLOCKING] D7's registry staleness rule cannot tell a stale entry from a live one

**Location:** `_lyx/discussion/decision-record.md` D7, second paragraph, and the Death Acceptance criterion.

**Issue:** D7 says registry evidence counts only when "not older than the newest `SessionStart` line: an entry whose process started before that line is stale".
A process always starts before the `SessionStart` line its own hook writes, so read literally every entry is stale, the live launch's included, and the registry half of D7 never fires.
Read loosely as "the entry belongs to the launch that wrote the newest line", it fails the case it was added for.
Before a resumed launch's `SessionStart(resume)` lands, the newest line is the crashed launch's own `SessionStart(startup)`, so the crashed launch's dead entry counts and fails the resumed launch.
That is exactly the "a stale crashed entry before a resumed launch's `SessionStart(resume)` ends nothing" criterion.
The comparison is also not stated in comparable units: `procStart` is clock ticks since boot (`internal/proc/proc_linux.go` `StartTime`, read by `claudeengine/prober.go` `processLiveness`), while a signal's time is wall-clock.
The next sentence, "Before any `SessionStart` line has landed for a launch, only a `SessionEnd` signals death", would cover the resume window on its own, but only if "for a launch" is defined, and the record never says how the liveness tick knows which lines belong to the current launch when, per D4's bound, an earlier launch's lines share the events file.

**Fix:** Replace the procStart-against-line comparison with a launch boundary the tick can test, such as the events offset recorded at this launch, and state the rule in those terms: registry death counts only once a `SessionStart` line past this launch's offset has landed.
Keep the resume criterion and add one for an entry of the current launch reading dead.

### [BLOCKING] D8 amends the PATTERN line's Enter rule without recording it

**Location:** `_lyx/discussion/decision-record.md` Constraints, `PATTERN-pane-text-injection` entry; D8's bound.

**Issue:** D8's bound says "A real character that survives the clear and the settle is submitted at the head of the message", and its residue path sends "as for the lone `%`".
That merges lyx's text with a box character and submits it, against the background's first sentence (never merges with or submits an operator's draft) and the `PATTERN.md` line "no Enter of lyx's own into a box not holding exactly the sent text".
The Constraints entry amends only "never wipes an operator's draft" and the key-delivery list, and states the `PATTERN.md` line "is unchanged".
The background already names the lone `%` as an admitted merge; D8 adds a second admitted merge that neither the background nor the line will name.
Under part A of the constraints every PATTERN breach is BLOCKING; this one is a breach unless the record carries the amendment.

**Fix:** Extend the Constraints entry: the background's `%` sentence gains D8's residue reading of a lone stray-character candidate, with its bound (cleared once, settled, persisting), and say whether the `PATTERN.md` line needs the same qualifier the `%` case relies on.
The alternative is to drop the residue submission and hold a persisting character as a draft.

### [MEDIUM] An escaped permission dialog leaves the run asking

**Location:** `_lyx/discussion/decision-record.md` D5 and the second Open risk.

**Issue:** D5 turns `PermissionRequest` into an ask at once.
In the fold (`internal/shuttleengine/sessionstate.go` `applyTranscriptMarkers`), the interrupt marker releases only a busy turn state, never an ask, and `judgeReadiness` holds an ask with cause `ask` as "the session waits on an answer".
An operator who presses Esc on a permission dialog interrupts the turn: no `Stop` follows, and the state stays asking until a new `UserPromptSubmit`, which lyx's own held sends cannot produce.
Today a dialog answered within the 6 s before `Notification(permission_prompt)` fires leaves a busy turn that the interrupt marker turns into `idle-stalled`, which is ready.
The Open risk covers only a granted permission and claims "sends are held either way", which is untrue for this path.
D5 states no bound on what reading asking at once now holds.

**Fix:** Decide in D5 that the interrupt marker past the ask releases it, as it releases a running turn, or record the escaped-dialog hold as a bound in D5 with its way out; add a criterion either way, and align the Open risk with D5.

### [MEDIUM] D7's registry veto and the fold's dead reading disagree

**Location:** `_lyx/discussion/decision-record.md` D7, first bullet and bound.

**Issue:** D7 says "a `SessionEnd` beside a registry entry whose pid is alive with a matching start time does not" end the run.
The fold's `settle` (`sessionstate.go`) puts a process-ending `SessionEnd` first, ahead of liveness, so the fold reads that same session dead.
`judgeReadiness` then holds every send as "the session's process is gone", `waitshadow.go` and the orch watcher compare against dead, and the run keeps waiting.
The record never says whether the fold adopts the veto.

**Fix:** State in D7 whether the fold's precedence changes so a live registry entry outranks a process-ending `SessionEnd`, or why the two readings may differ, and add the matching fold criterion.

### [MEDIUM] D1 and D3 disagree on whether hook order matters

**Location:** `_lyx/discussion/decision-record.md` D1 last paragraph, D3 bound, Constraints last bullet.

**Issue:** D1: "If the corpus shows no `SessionStart(startup)` line from the `--settings` file before the first typed turn, D3's `startup` rule and D4 cannot hold."
D3's bound handles a `UserPromptSubmit` landing before `SessionStart(startup)`, and Constraints says "D3 already holds for either hook order".
Either the order matters to D3 or it does not; the plan writer cannot tell which condition the capture must test.
Under D4 lyx types no turn before the line, so "before the first typed turn" also cannot be observed in a D4 world; the capture has to state what it measures.

**Fix:** Restate D1's condition as what D3 and D4 depend on, presumably that the `--settings` `SessionStart(startup)` line arrives at all and before the pane's ready reading, and keep "either hook order" only where it is true.

### [MEDIUM] D1's blocker leaves the rest of the task's fate open

**Location:** `_lyx/discussion/decision-record.md` D1, last paragraph; Acceptance criteria.

**Issue:** If the corpus contradicts D3's startup rule, the implementer "builds neither" it nor D4.
The record does not say whether the batch stops, or whether D2, D3's `resume` and `clear` sources, D5 to D9 and the other criteria proceed.
The #549 criterion depends on D3's startup rule and is stated unconditionally, so a plan that follows D1 cannot meet it.

**Fix:** Say what the plan builds under the blocker: which Decisions proceed and which criteria become conditional on the corpus.

### [LOW] D8's wipe window has no stated limit

**Location:** `_lyx/discussion/decision-record.md` D8 bound.

**Issue:** "An operator loses ... anything typed on that line between the guard reading and the clear."
The background this amends promises never to wipe a draft; the exception is bounded in which box triggers it but not in how long the window between the reading and the clear can be, and the clear removes the whole line, not one character.

**Fix:** State the window's bound, such as the clear going out right after the guard reading with no intervening wait, or an operator-activity re-check immediately before the clear.

### [LOW] Two criteria do not match their Decisions

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, Death and Startup bullets.

**Issue:** "A dead process before startup is ready fails the start as died" conflicts with D7's "Before any `SessionStart` line has landed for a launch, only a `SessionEnd` signals death": a crash before the line writes no `SessionEnd`, and reed's login-shell pane outlives `claude`, so the start times out instead.
D4's bound, that an earlier launch's `SessionStart` line satisfies the hook half and the pane still gates, has no criterion.

**Fix:** Narrow the death criterion to the evidence D7 accepts, or extend D7; add the earlier-launch-line criterion under Startup.

### [LOW] Q2 contradicts the record

**Location:** `_lyx/discussion/support-log.md` Question ledger, Q2.

**Issue:** Q2 answers "Either way a `SessionStart` line proves trust passed" and "else D4 escalates (Open risks)".
The record makes the trust proof conditional on the corpus (D4 rationale, third Open risk) and moved the escalation into D1, as the Turn 8 entries say.

**Fix:** Update Q2's answer to the conditional and point it at D1.

### [NIT] Restatements outside the Decisions

**Location:** `_lyx/discussion/decision-record.md` Auto-mode assumptions, third Open risk, last Constraints bullet.

**Issue:** The second and third Auto-mode assumptions restate D1's capture ("D1 confirms it", "D1 records both").
The third Open risk restates D4's conditional rationale.
The last Constraints line binds another board entry's future work and restates D3's bound; this plan cannot act on it.
None carries a requirement absent from its Decision.

**Fix:** Cut them, or move the deliberation to `support-log.md`; the requirements already live in D1, D3 and D4.
