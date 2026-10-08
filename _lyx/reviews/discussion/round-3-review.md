---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:138
    summary: D7 says the stop mark is the one Run field shared with the Wait goroutine and that nothing else takes the new mutex, but the stop verb's record write shares run.state and run.json with Wait's own saves
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:68
    summary: D3 lets a confirm read after the window closes send another Enter while also requiring one confirm read per Enter, and leaves the sleep before that last read at a grown interval, so the stated overshoot bound does not hold
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:166
    summary: Constraints audit the stopped value at one return in the stop verb, but D7 has finalize persist stopped for a handle-marked run too, a second write the tripwire as described never sees
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:59
    summary: D2 cuts the appearance check and its replay short at the window close but never says which error a text that never appeared returns, so D4 and D6 cannot tell whether it is retried, and the clear does not cover a doubled replay
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:74
    summary: D3 clear rests on D1 proving the box empty, which holds only for an engine with the idle reading, and a prefix with no minimum length or any paste placeholder can delete a foreign draft the open risk does not describe
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:112
    summary: D6 restores a cleared boundary without saying the gate is re-entered on that tick when nothing is pending, whether the unsent re-prompt mark is restored, or that the loop must read the pane every tick to time the 10 s override
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:169
    summary: Constraints call isTerminalOutcome attach's only change, but Scope and D7 also move attach's superseded-strand removal onto the stop verb
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:198
    summary: D1 acceptance criterion says the pane has read idle for 10 s where D1 and the D6 criterion say idle on every poll for 10 s
---

### [BLOCKING] D7's mutex claim does not cover what the stop verb's record write shares with Wait

**Location:** `_lyx/discussion/decision-record.md` D7, the paragraph beginning "The mark is the one field of `Run` shared".
**Issue:** D7 states that the mark is the one `Run` field shared between the stopping goroutine and the `Wait` goroutine, and that the new mutex is taken only by the stop verb and by finalize.
The code contradicts this.
`saveRunState(run.runDir, run.state)` writes the whole in-memory `RunState` by value, and `Wait`'s goroutine calls it outside finalize: `hold.go:78` persists `NotifiedOffset` on each held-turn notice, and `wait.go:901` persists `Started`.
It also reads `run.state.SessionID` and `run.state.StrandGUID` on every tick.
D7 does not say what the stop verb writes the record from.
Two readings, both unsound as written:

- If the verb sets `run.state.Outcome` and saves, it writes a field of a struct the `Wait` goroutine copies concurrently without the mutex, which is a data race, and the claim "one field shared" is false.
- If the verb reads `run.json` from disk, patches `Outcome` and writes it back, a later `hold.go` or `wait.go:901` save from `run.state` restores `running` over `stopped` until finalize rewrites it, and the verb's patch can drop a `NotifiedOffset` update the other way.

The acceptance criterion "A burler half stopped during `join` ... reads `stopped` after its own `Wait` returns" holds only because finalize rewrites last; the intermediate state D7's rationale relies on (a terminal record before the strand goes) is not guaranteed.
**Fix:** State the record's single write path for a run held in-process: either the verb mutates `run.state` and every `saveRunState` call on `run.state` from the `Run` takes the same mutex, or the mutex is documented as a record-write lock that all of them take.
Replace "one field" and "nothing else takes it" with the real set.
Keep the guid form (no `Run`) as a disk read-modify-write and say that it has no in-process `Wait` to race.

### [MEDIUM] D3's last confirm read breaks its own loop and its own overshoot bound

**Location:** `_lyx/discussion/decision-record.md` D3, the paragraph beginning "After Enter, the input box is re-read".
**Issue:** Three sentences disagree.

- "Each read that shows the box still holding the sent text sends one more Enter" has no window condition, so the one confirm read taken after the window closes, if it still shows the text, sends an Enter.
  "Every Enter is followed by one confirm read" then requires yet another read, and so on.
  The text never says a read taken after the window closed sends no Enter.
- "The box is re-read at a growing interval, starting at `submit_redraw_settle_ms` and doubling up to 5 s" does not say whether the sleep after an Enter is the grown interval or always `submit_redraw_settle_ms`.
  The overshoot sentence claims "one `submit_redraw_settle_ms` sleep and one capture", which is true only for the second reading; with the first, an Enter sent just before the close overshoots by up to 5 s.
- D6's "at most its tail past the window" lists the confirm read, the clear and its two readings, and D5's capture.
  It omits the final poll sleep and read of D1's idle wait, which also ends by the deadline, and the clear's own choreography sleeps.

D2's settle reads are inside the window and a box not settled at the close gets no Enter, so the "a send never fails without reading the box after its last Enter" guarantee holds there trivially; the loop after the first Enter is where it needs the rule.
**Fix:** Say that a read taken after the window closed sends no Enter and ends the loop.
Say that the sleep before each post-Enter read is `submit_redraw_settle_ms` for the read an Enter triggers, and the doubling interval spaces the Enters.
In D6 list the overshoot as the union of these tails, naming the idle wait's last poll.

### [MEDIUM] Constraints audit one site, but D7 writes `stopped` from two

**Location:** `_lyx/discussion/decision-record.md` Constraints, `PATTERN-completion-signal` bullet, against D7.
**Issue:** The Constraint makes `runOutcomeStopped` appear at one return in the stop verb, adds the identifier to `negativeVerdictMarkers` and the verb's file to the scanned files.
The scan keys on identifiers in return statements.
D7 also has a handle-marked run's own `Wait` finalize persist `stopped` "for every outcome but `done`".
`finalize` sets `run.state.Outcome` by assignment (`wait.go:1187`), not at a return, so a mark-driven substitution there would be invisible to the tripwire even though it is a second negative answer to "did this finish".
The Constraint is silent on how finalize obtains the value.
**Fix:** Require that finalize take the stored outcome from the same one function the Constraint names, so the value appears at one return and finalize calls it.
State which `allOutputFilesExist` call guards the finalize path, or that the `done` precedence in D7 is that guard.

### [MEDIUM] A text that never appeared has no stated error class

**Location:** `_lyx/discussion/decision-record.md` D2, the sentence "both run inside the submit window of D3, so a replay is cut short when the window closes".
**Issue:** D2 and D3 name `ErrSubmissionNotLanded` only for an unsettled box and for a submission pending at the window's end.
The existing appearance check ends with a plain error ("sent text never appeared in the pane ... NOT delivered", `run.go:1096`), and D2 now also ends it early when the window closes.
D4 retries only the two sentinels and D6 ends the loop on "any other re-prompt send error", so whether this failure is a retried infrastructure miss or a halt is left open.
Its cause (the TUI swallowed the input) is the same class D6 calls "not a spent attempt".
A replay that did type the text twice can also leave two copies in the box, which D3's clear ("the sent text, a prefix of it or a paste placeholder") does not name.
**Fix:** Decide the class: either that failure wraps `ErrSubmissionNotLanded`, or it stays plain and D6 and D4 say so.
If it wraps the sentinel, extend the clear's matching rule to a box holding the text repeated.

### [LOW] D3's clear is bounded by a premise that holds only for some engines, and its match rule is wider than the open risk says

**Location:** `_lyx/discussion/decision-record.md` D3, the clear paragraph, and Open risks.
**Issue:** "D1's idle wait proved the box empty before typing, so such content is this send's own" holds when the engine has `IdleSession`.
The clear is gated on the engine owning a clear choreography and on a box reading, that is `InputBoxReader`, not on the idle reading; D1 gives an `InputBoxReader` engine without `IdleSession` only `requireReadyAgentPane`, which does not prove an empty box.
The match rule also has no floor: any prefix of the text, including one character, and any collapsed paste placeholder qualify, so a foreign draft starting alike or an operator's own paste is deleted.
The Open risk covers "text matching it" only, which does not describe the placeholder case.
**Fix:** Tie the clear to an engine that supplies both the idle reading and the clear, or state the premise's limit.
Give the prefix a minimum length or say that any prefix counts, and reword the Open risk to cover a placeholder.

### [LOW] D6 leaves the restore's trigger and its per-tick reading unstated

**Location:** `_lyx/discussion/decision-record.md` D6, the paragraph beginning "A boundary cleared by a `turn_start` that has no later turn end".
**Issue:** The wait loop re-enters `handleGatedBoundary` without a new arrival only for `gatePending && gateAtBoundary` (`wait.go:276`), and D6 adds the unsent-re-prompt mark.
D6 says the boundary is "restored on the tick where D1's override releases it" but not that the gate is evaluated on that tick when nothing is pending and no mark exists, which is the case for a replayed first tick whose Done was cleared.
A turn start also "clears the boundary and the mark", and D6 does not say whether the restore brings the mark back.
The 10 s override counts pane reads "on every poll", and the wait loop reads no pane per tick today, so D6 adds a per-tick capture and a timer that the Decision does not name.
**Fix:** State that the restore evaluates the gate on that tick, whether the mark is restored or the evaluation re-derives the re-prompt, and that the loop reads the pane each tick while a boundary is cleared.

### [LOW] Constraints understate attach's change

**Location:** `_lyx/discussion/decision-record.md` Constraints, last sentence of the `PATTERN-completion-signal` bullet.
**Issue:** "`isTerminalOutcome` recognising `stopped` is attach's only change" disagrees with Scope and D7, which move attach's superseded-strand removal (`removeSupersededStrands`) onto the stop verb.
`isTerminalOutcome`'s own doc comment ("one of the values finalize ever wrote") also stops being true.
**Fix:** Say it is the only change to attach's classification, and list the removal call as the other.

### [NIT] D1's criterion drops "on every poll"

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, the `turn_start` criterion.
**Issue:** It reads "until ... the pane has read idle for 10 s", where D1 and the D6 criterion require idle on every poll for 10 s.
**Fix:** Use the D1 wording.

## Focus verdicts

- `ErrSubmissionNotLanded` and the typed text: D3 clears it, only on a box holding this send's text, with a post-clear reading; D6's retry cannot pass D1's idle wait while text remains, because D1's reading fails a non-empty box. Residual gaps are F4 and F5.
- D1's interrupt and 10 s override apply to a boundary cleared by an unmatched `turn_start` (D6 says so), and the two acceptance criteria agree except for F8; the trigger is F6.
- Deadline overshoot: the window closing by the deadline bounds it and the appearance check sits inside the window; a read after the last Enter is guaranteed only as far as F2 repairs.
- Burler stops a half through the handle form, so the stop and `Wait` meet in one process; where the mutex lives and what it covers is F1.
- The marker is the `runOutcomeStopped` identifier at a return; the finalize write is F3, and the attach statement is F7.
