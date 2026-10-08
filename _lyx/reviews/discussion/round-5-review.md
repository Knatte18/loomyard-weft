---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 Bound and D2 Rationale
    summary: New hook lines and stamp lines bump the events file mtime, which batten's ReadAgentActivity counts as LastActivity, so the quiet notice shifts and shadow mode is not behavior-neutral
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 stamp pairing and D3 wait loop fold per poll
    summary: A poll that lands between a stamp line and its payload line consumes the stamp alone, so the payload gets an empty since for that reader only, contradicting the same-for-every-reader claim
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 history list resting only on signals
    summary: A waiting turn end whose tasks come from the transcript fallback rests on the transcript read at parse time, so it is per reader, not signal-only
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints PATTERN-completion-signal line, D3 idle-done paragraph and precedence rules 2 and 3
    summary: The record says every negative state consults output files first, but precedence rules 2 and 3 and the acceptance rows put dead and unknown liveness-unproven above a turn end with every output present
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3 busy and turn start entries
    summary: Turn starts are listed as signal-only, yet the newest turn start reads idle-stalled interrupt in a reader that marks it and busy in one that does not, and record-on-change couples neighbouring entries to that difference
  - id: F6
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md D1 Rationale lines on cat versus printf chunking
    summary: The rationale carries the measured chunking comparison that is a rejected-alternative trail already in support-log; keep only the requirement and the acceptance row that pins it
---

### [MEDIUM] Hook and stamp lines change batten's activity reading in shadow mode

**Location:** `decision-record.md` D1 Bound (the wait loop's input changes only by the stamp lines it skips), D2 Rationale (shadow mode must change no behavior), D5 (batten's activity reading logs only).
**Issue:** `internal/shuttleengine/activity.go` (`readRunActivity`) sets `LastActivity` to the newer of the transcript mtime and the events file mtime, and batten's quiet notice compares the newest `LastActivity` against its threshold (`battenshed/notice.go`).
Every new hook line and every stamp line is a write to the events file, so each one moves that mtime.
A `Notification` `idle_prompt` is the clear case: it fires after the session sits idle, when the transcript no longer changes, so the append restarts batten's quiet clock for a stalled session.
The Bound speaks only of the wait loop's input and says nothing of this second reader, so the claim that shadow mode changes no behavior is false for batten, which the record names as one of the three sites.
**Fix:** Decide the question in D1: either bound it (the quiet notice moves later by at most one idle threshold per idle stretch, accepted in shadow mode, with an acceptance row), or keep `ReadAgentActivity` from counting lines `ParseEvents` skips, for example by reading the mtime of the last parsed Stop or ask line.
Extend the Bound to name every reader of the file's mtime or size, not just the wait loop.

### [MEDIUM] A cross-poll split of stamp and payload gives a reader-dependent `since`

**Location:** `decision-record.md` D1 (stamp pairing by the signal parser, "the same for every reader"), D3 (the wait loop folds each poll's new signals).
**Issue:** The stamp and the payload are two appends, a few process spawns apart (`date`, `printf`, then `cat`).
`readEventsFrom` consumes every complete line past its offset, and a stamp line is complete on its own.
A poll that falls between the two appends hands the stamp to one fold and the payload to the next.
The next batch has no preceding stamp, so the payload takes an empty `since` for the wait loop and the orch watcher's cursor, while a whole-file reader pairs it correctly.
D1 states that a payload without a stamp has an empty `since`, the same for every reader; this case makes it differ.
The acceptance rows pair two stamps of different hooks in one batch and never test a split batch.
**Fix:** State where the pending stamp lives for an incremental reader, for example the incremental reader does not advance its offset past a stamp line that has no payload after it, or the parser input always starts at the prompt offset.
Add an acceptance row that splits the batch between a stamp and its payload.

### [MEDIUM] Waiting turn ends from the transcript fallback are not signal-only

**Location:** `decision-record.md` D3, the paragraph beginning "The entries resting only on signals (turn starts, waiting turn ends, ...)".
**Issue:** `outstandingBackgroundTasks` in `claudeengine/background.go` uses the Stop payload's `background_tasks` when present and otherwise reads the whole current transcript (`transcriptHasUnmatchedLaunch`) at parse time.
A turn end whose tasks come from that fallback therefore depends on the transcript as it is when the reader parses.
An older turn end re-read after the task's completion notification landed in the transcript loses its outstanding tasks, so a late files-only reader records `idle-done` or `no-output` where the wait loop recorded `busy` `background`.
That is a transcript-resting entry, exactly the class D3 files under per-reader.
D2 also lets the signal parser return "turn end (with its outstanding background tasks and their reporting signal)", which hides that the fallback is a reading-time read.
**Fix:** In D3 limit the signal-only claim to waiting turn ends reported by the payload, and list a waiting turn end reported by the transcript fallback with the per-reader transcript markers.
Say in the acceptance rows that a late reader may read such a turn end without its tasks.

### [MEDIUM] Precedence rules 2 and 3 contradict the output-files-first statements

**Location:** `decision-record.md` Constraints (`PATTERN-completion-signal`: every negative state consults the output files first), D3 `idle-done` paragraph (Output files are consulted before any negative state), precedence rules 2 and 3, acceptance rows for precedence.
**Issue:** Rule 2 gives `dead` `process-gone` after a turn end with every output present, and rule 3 gives `unknown` `liveness-unproven` there.
The acceptance rows require both.
The Constraint and the D3 sentence say the output files come first for every negative state.
A plan writer cannot satisfy both.
`PATTERN-completion-signal` itself covers the shuttle outcomes (`OutcomeDied`, `OutcomeTimeout` and the like), not this reducer's states, so the Constraint overstates the pattern; the stated ordering is a deliberate choice that the record does not mark as one.
**Fix:** Narrow the Constraint and the D3 sentence to the output-derived states (`no-output`, interactive `asking`), and state that `dead` and `unknown` are process facts ranked above them in shadow mode because no consumer acts on them.
Add that a first consumer must apply `PATTERN-completion-signal` before acting on `dead`.

### [LOW] Turn start entries are per reader once an interrupt is marked

**Location:** `decision-record.md` D3, `busy` and the history paragraph.
**Issue:** `busy` reads `idle-stalled` instead when the interrupt marker follows the newest turn start.
A reader that marks the marker records `idle-stalled` `interrupt` for that turn start, and a reader that folds later, after a new prompt, records `busy`.
The record-on-change rule then couples the next signal-only entry to that difference: a repeated turn start after an interrupted one is recorded in one history and coalesced in the other, since a turn start gives no new state or cause.
The listing of turn starts among the entries that are the same for every reader is therefore wrong at the edges.
**Fix:** Say that the newest turn start's entry is per reader, and state the cause a turn start gives to `busy`, so the coalescing rule has a defined answer.

### [NIT] Chunking measurement is a rejected-alternative trail

**Location:** `decision-record.md` D1 Rationale ("A separate stamp line leaves the payload's write unchanged: `cat` writes ... 4096-byte chunks ...").
**Issue:** The measured comparison of `cat` against a shell `printf` explains why the one-line `printf` of the payload was rejected, and `support-log.md` already carries it under Rejected alternatives.
The requirement is already in the Decision (the payload by today's command, unchanged) and the acceptance row pins it with a 20000-byte payload.
Completeness check: the plan writer needs the 20000-byte reason only to understand the row, which the row's own text ("past the shell's 4096-byte `printf` chunk") already gives.
**Fix:** Cut the measurement sentences from the Rationale and keep the one-line reason that the stamp is a separate short append so the payload's write is unchanged.

## Verdicts on the focus items

Each item below was reached and judged.

1. **Hook append in D1.**
   The payload `write` is unchanged, since the payload still goes through `cat`, so the 4096-byte chunking concern does not apply to the Stop and ask lines.
   The Bound as written (byte-identical lines, same command, same `ParseEvents` result) holds, and it does not claim the absence of concurrency risk: the later paragraph and the Open risk accept the gap between a payload and its newline, now with more appenders.
   The Bound and the acceptance row state the same behavior.
   What the focus did not name but the reading found is F1 (mtime readers) and F2 (split batch).
2. **`dead` and `SessionEnd` `clear` or `resume`.**
   Not a defect.
   Rule 1 names the case and routes it to `unknown` with the reason under rule 5, and the acceptance rows state it with a live process.
   Claude Code 2.1.294's reason list, read from the binary, is exactly `clear`, `resume`, `logout`, `prompt_input_exit`, `other`, so D3's classification is complete.
   The one nuance is that rule 3 (`liveness-unproven`) outranks the `clear` or `resume` cause when the registry entry does not follow the session; both are `unknown`, and D4 and the Open risks already say so.
3. **The orch watcher's session id.**
   The premise that the orch has no run record does not hold: `Runner.ReadEvents` resolves the orch's strand through `FindRun`, and `orchengine/doc.go` documents the per-launch sentinel output file.
   D3 therefore gets the spawn-time id from that record and the current id from the newest hook line past the prompt offset, and the acceptance row for the `/clear` lookup states the choice generally.
   Not a defect.
4. **Signal-only history entries.**
   Defective at the edges: `interrupt` and a transcript-marked `api-error` are already filed as per reader, but the waiting turn end from the transcript fallback and the interrupt-affected turn start are not.
   See F3 and F5.
5. **D3 against D6 on the bounding entry.**
   Not a defect.
   Both Decisions point at the newest history entry before the call-time state that is not itself a call-time state; "bounds how long it has held" and "shows when the state last changed otherwise" describe that one entry from two sides.
   The entry can itself be per reader (an `idle-done` judged by a stat), which D3 already says, and where no entry precedes the call-time state nothing is shown, which is the correct reading.
