---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3, Open risks, Acceptance criteria
    summary: D3 requires a clear that empties a multi-line restored prompt, but the only clear Claude has is one C-u shared with the stray-character and sent-text clears, and the record leaves the sequence choice, its bound and its test open
    fix: In D3, replace the sentence that starts with The clear sequence must empty a multi-line restored prompt with a decision. Either D3 sends its own restored-prompt clear sequence, declared in claudeengine beside ClearInputSequence, which stays unchanged, or ClearInputSequence itself changes. Then state that sequence's bound in D3's Bound line, for example that it holds no Escape, C-c or Enter and only deletes box text. In Acceptance criteria, add that a multi-line restored prompt is emptied by creel's interrupt clear on the shuttle fake. Reword the second Open risks bullet to the risk that remains once the choice is made.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3, Auto-mode assumptions
    summary: D3 reads the box once after InterruptSettle, so it rests on Claude redrawing the restored prompt within the settle, an assumption missing from Auto-mode assumptions and with no stated outcome when it fails
    fix: Add a D3 bullet to Auto-mode assumptions saying that Claude draws the restored prompt into the box within its InterruptSettle, and that the plan confirms this from a live capture. In D3, state what happens when the prompt appears after the post-settle read. The worker resumes, and its pre-write guard reads either an empty box or a draft. Name what bounds a paste that lands before a late restore.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 Bound
    summary: D1's bound says every non-blank interior cell carries the placeholder style, but the box interior also holds the caret and side bars, which Claude does not draw in that style, so the bound read literally never blanks; the lone % end-of-line mark beside a placeholder is also left unstated
    fix: In D1's Bound line, change every non-blank interior cell to every non-blank cell of the box's text once the caret and side bars are stripped. Also state whether a lone % beside placeholder-styled text reads as empty or as a draft.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D5
    summary: D5 picks the client's start header by time, so a concurrent client's later spawn can be quoted as this start, and the bound claims truncation loses nothing of a start that has not happened, though another client's truncation can erase a start that already did
    fix: In D5, have the client select the start header by the pid of the process it spawned, which the header already carries, not by time. Rewrite the last clause of the Bound line to say that a concurrent client's truncation past the cap can erase this start's lines, and that the give-up error then says the file holds no line from that start. In Acceptance criteria, change never a line from an earlier start to never a line from another start.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2
    summary: D2 sends a progress line on the open inject --wait connection without saying whether the CLI prints it, while every creel verb prints one JSON envelope on stdout
    fix: In D2, state that the progress line is a wire message the client keeps, and that the verb still prints one envelope, carrying the last reason and its time as fields when it ends expired or timed out. If the progress is meant to be shown live, name stderr as its stream.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Several decided behaviors have no criterion that would fail without them, namely D2's expired record carrying the reason and its too-short way-forward command, and D8's privilege wording in help text and the resolved sender on non-waiting verbs
    fix: In Acceptance criteria, extend the D2 criterion with an expired record that carries the stall reason, and a too-short reason that names an allowed lyx command. Extend the D8 criterion with creel help text that states the privilege rule as doc.go does, and with every verb that prints a sender printing the resolved one.
---

### [MEDIUM] D3's multi-line clear is required but undecided

**Location:** `_lyx/discussion/decision-record.md` D3, Open risks, Acceptance criteria

**Issue:** `ClearInputSequence` in `internal/shuttleengine/claudeengine/session.go` is one `C-u`.
Its own comment says that key deletes only from the caret to the start of the line, which is enough for a single line.
It is the same sequence the stray-character and sent-text clears send under `PATTERN-pane-text-injection`.
D3 says the sequence must empty a multi-line restored prompt, so the current one does not meet D3 as written.
The record then hands the choice to the plan ("the plan checks", "the plan finds the sequence").
If the plan widens the shared sequence, it changes the bounded stray clear without a bound stated anywhere.
That is the guard-weakening case the attack-surface rule and the spec's compaction rule require a Decision to bound.
No acceptance criterion would fail if a multi-line prompt were left behind.

### [MEDIUM] D3's post-settle read assumes when the restored prompt appears

**Location:** `_lyx/discussion/decision-record.md` D3, Auto-mode assumptions

**Issue:** D3 reads the box once after the settle and clears only text it sees then.
It then wakes the worker, whose next probe may find the box empty.
If Claude draws the restored prompt after that read, D3 never clears it.
The worker can also pass its pre-write guard on the empty box before the restore, and Claude's paste-submit sends its Enter in the same tmux command.
The D3 auto-mode assumption covers only early versus late interrupts, not timing.
This is one of the two assumptions the focus asked to be checked.

### [LOW] D1's bound counts cells the placeholder never covers

**Location:** `_lyx/discussion/decision-record.md` D1 Bound

**Issue:** `inputBoxInterior` returns whole lines between the rules, caret and side bars included.
`InputBoxText` and `holdsOnlyEOLMark` strip those characters before reading.
D1's main text says "a box interior whose text is drawn wholly in the style", which is right.
The Bound line says "every non-blank interior cell", which taken literally includes the caret, so no box would ever blank.
The record also does not say how the `%` end-of-line mark, which reads as empty today, combines with placeholder-styled text.

On the focus question: the bound holds for the landing check.
A wholly placeholder-styled box reads empty there, which never equals the sent text, so no Enter of lyx's own follows.
A box mixing typed and placeholder text, and the collapsed-paste shape, are never blanked, so the landing check sees what it sees today.

### [LOW] D5's start match and truncation bound

**Location:** `_lyx/discussion/decision-record.md` D5

**Issue:** `spawnDaemon` in `internal/creelcli/spawn.go` starts the daemon through `exec.Cmd`, so the client holds its child's pid.
Picking "the newest start header written at or after its own spawn" can select a header from a daemon another client spawned a moment later.
Another client can also truncate the file once it exceeds the cap, after this client's daemon has written its error.
The Bound says truncation "loses nothing of a start that has not happened", but the start already happened in that race.

### [LOW] D2's progress line has no stated output stream

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** `injectCmd` in `internal/creelcli/cli.go` returns one result that `runVerb` prints as one JSON envelope.
D2 adds a progress line on the connection and a final reason print, but does not say whether progress lines reach stdout.
Lines printed there before the envelope would break callers that parse a single envelope.

### [LOW] Acceptance criteria miss some decided behaviors

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** Checking each Decision against the criteria:
- D1, D3, D4, D6, D7 and D9 each have a criterion that would fail without them.
- For D2, nothing checks the `expired` record's reason or the too-short reason's way forward, the second of which is a `PATTERN-no-denied-recovery` constraint.
- For D8, nothing checks the help-text privilege wording or the resolved sender on non-waiting verbs.

No criterion asserts something no Decision decides.

## Notes

- D2 and D3 agree on the stall record during an interrupt: D3 suspends the worker's probe, send and stall record, and D2's draft reason is journaled by the worker's first probe after the interrupt wakes it.
- D8's `release` is bounded to the caller's own live hold, and its unknown-name refusal matches `hold`'s, as `internal/creelengine/doc.go` states the receiver rule.
- No Decision edits the task's own board entry, and no finding relies on a `Do not flag` item.
