---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/editrecord.go:68
    summary: The edit hook adds a frequent concurrent appender to events.jsonl, and an edit write landing between another recording hook's payload append and its newline append merges both into one malformed line, losing that hook's signal such as a Stop turn end.
    fix: Stop appending edit records to events.jsonl. Append them to their own file in the run directory, beside events.jsonl, with its name declared once as a constant. The settings' edit hook names that file, and RecordEdit writes the same stamp-and-record pair there. watchEditLoops reads that file through the engine's ParseSessionSignals with its own cursor. The cursor starts at RunState.EditLoopNotifiedOffset, now an offset in that file. Update the RunState field comment, the shuttleengine/doc.go and claudeengine/doc.go paragraphs, editrecord.go's comments and the summary to name the file. Point TestWait_EditLoopNotice's appendEvents and withRunEvents seeding at the new file, and TestBuildSettings_EditHook's expected command at the new path.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: internal/shuttleengine/editloop.go:90
    summary: The board entry asks for a friction note to the run's parent, but the diff sends a Shuttle notice line through the notifier and writes no friction note, a reinterpretation no Added after Discussion entry records.
    fix: Leave the code in place and defer this finding to the run's parent. Recording or refusing the reinterpretation of friction note as a notifier line, with lym loom decision add, is the parent's call and lies outside the fixer's write surface.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/editloop_test.go:876
    summary: No test pins that fold counts an edit at its hook-side stamp time, because the fake engine leaves At zero without markEvents, so every row's edits take the run clock and a regression ignoring signal.At passes.
    fix: In TestWait_EditLoopNotice, build the sessionFakeEngine with markEvents set for two new rows whose events carry stamps. In the first, three EDIT lines of a.go are stamped more than the one-minute window apart but read in one batch; it wants no notice. In the second, three stamped within the window are read only after the run clock has moved past the window; it wants one notice. Use whatever stamp syntax fakes_test.go's markEvents branch reads.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/editloop.go:104
    summary: When persisting EditLoopNotifiedOffset fails, the cursor has already advanced and fold has already deleted the loop's key, so the notice is lost for good, where the dialog notice retries on a later tick.
    fix: In watchEditLoops, keep the cursor value from before the read and a clone of run.editLoops.edits taken before fold. On a persistEditLoopNotifiedOffset error, restore both, so the next tick re-reads the batch and retries the notice, as notifyDialog does. Add a TestWait_EditLoopNotice row whose state save fails on the first persist and succeeds after, wanting one notice on the later tick.
  - id: F5
    severity: LOW
    class: decision
    origin: lens:code-scope
    location: internal/shuttleengine/engine.go:319
    summary: Only the claude engine implements EditRecorder, so a codex run never gets an edit-loop notice, although the board entry asks shuttle to count edits without naming an engine.
    fix: Leave the code in place and defer this finding to the run's parent. The parent either records the claude-only scope with lym loom decision add, or asks for a codex EditRecorder and edit hook. Either is the parent's call.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/editloop_test.go:806
    summary: The expected notice is built by editLoopNotice itself, so no row pins that the line carries the file, the subagent id, the count and the window.
    fix: In TestWait_EditLoopNotice, check each received notice for what it must carry, with strings.Contains or a literal. It holds the strand name, the file path inside the notice delimiters and the count. A subagent row also holds the agent id inside the delimiters. The window holds as time.Minute's string form. Drop the self-built expectation, or keep it beside the checks.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/editrecord_test.go:349
    summary: No row covers RecordEdit returning an error when the events file cannot be opened.
    fix: Add a TestRecordEdit row whose events path lies under a directory that does not exist, so it cannot be opened. It wants an error naming append the edit record or open the events file. Let the row set its own events path in place of the t.TempDir default.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:662
    summary: The overview lists an autonomous run's parent notices and the PostToolUse hooks with their hidden verbs, but leaves out the new edit-loop notice, the hook-edit hook and the edit_loop_edits and edit_loop_window_s keys.
    fix: In docs/overview.md, extend the sentence at line 662 with a third notice, once per edit loop, for one file edited edit_loop_edits times within edit_loop_window_s. Beside the hook-batching sentence at line 703, add one sentence. While edit_loop_edits is above 0, an autonomous claude run's Edit and Write calls run the hidden lym shuttle hook-edit, which records the edited file for that watch.
---

### [MEDIUM] Concurrent edit-hook appends can split another hook's events line

**Location:** `internal/shuttleengine/claudeengine/editrecord.go:68`, with `internal/shuttleengine/claudeengine/settings.go:395-398`.

**Issue:** `recordingCommand` appends a payload with `cat >>` and its newline with a separate `printf '\n' >>`.
Before this diff, no frequent writer ran beside those recordings.
Now every Edit or Write of a background subagent or fork appends to the same file while the main session's Stop, Notification or UserPromptSubmit recording may be mid-append.
An edit write between the two appends puts the stamp line on the end of the other payload's line.
`ParseSessionSignals` drops the whole merged line (`signals.go:104-107`), so a turn end can vanish, and the edit after it takes a zero time.
The edit hook's own single write is safe; the defect is the new interleaving it adds to the shared file.

### [MEDIUM] Friction note reinterpreted as a notifier line without a recorded decision

**Location:** `internal/shuttleengine/editloop.go:90`; `_lym/landing/summary.md`, Design decisions, third bullet.

**Issue:** The board entry's second idea point asks for "a friction note to the run's parent".
In this repository a friction note is a record type whose one writer is `internal/friction` (`PATTERN-run-records`).
The diff sends a `Shuttle notice:` line through the runner's notifier instead.
The summary explains the reading, and it is a defensible one, but the record carries no `Added after Discussion` entry approving it, so the call is the parent's.

### [MEDIUM] Stamp-time counting is untested

**Location:** `internal/shuttleengine/editloop_test.go:876`.

**Issue:** `fold` counts each edit at `signal.At`, and the summary relies on that for the window and for edits read after a pause.
The test's `sessionFakeEngine` sets `At` only under `markEvents`, which the test leaves off.
Every edit therefore reads a zero time and takes the run clock, so no row would fail if `fold` ignored `signal.At`.

### [LOW] A failed offset persist drops the edit-loop notice for good

**Location:** `internal/shuttleengine/editloop.go:104`.

**Issue:** `watchEditLoops` advances `run.editLoops.cursor`, and `fold` deletes the completed loop's key, before the persist.
When `persistEditLoopNotifiedOffset` fails, it returns after one Warn.
No later tick can see the loop again.
`notifyDialog` handles the same failure by returning false and retrying.
No row covers either this path or a notifier error.

### [LOW] Codex runs get no edit-loop watch

**Location:** `internal/shuttleengine/engine.go:319`.

**Issue:** Only `Claude` implements `EditRecorder`, so a codex run installs no edit hook and never reads a file edit.
The board entry is engine-neutral, and the summary states the narrowing without a recorded decision.

### [LOW] The notice's content is asserted against itself

**Location:** `internal/shuttleengine/editloop_test.go:806`.

**Issue:** The expected notice is `editLoopNotice(...)`, the function under test, so dropping the file, the agent id, the count, the window or the delimiters still passes.
These are Go-filled values, not static prose, so asserting them is in bounds.

### [LOW] RecordEdit's write failure is uncovered

**Location:** `internal/shuttleengine/claudeengine/editrecord_test.go:349`.

**Issue:** `RecordEdit` wraps an open or write failure into an error that `hook-edit` reports on standard error.
No `TestRecordEdit` row reaches that path.

### [LOW] The overview omits the new notice, hook and keys

**Location:** `docs/overview.md:662`, `docs/overview.md:703`.

**Issue:** The overview lists the parent notices of an autonomous run (held turn end, dialog) and documents the sibling PostToolUse hook `hook-batching`.
It now misses the edit-loop notice, `hook-edit` and the two `shuttle.yaml` keys.

## Rejected

- code-correctness finding 3 (a reattach forgets partial counts before the persisted offset): the `RunState.EditLoopNotifiedOffset` comment states this ("counts no edit at or below it"). It is the cost of never repeating a notice. The notice is advisory and a reattach is rare. Persisting partial counts would add record state for little gain.
- code-correctness finding 4 (out-of-order times escape the front-only trim): `RecordEdit` writes the stamp and the record in one write, so an unstamped edit arises only from the line corruption F1 removes. Second-granularity stamps of parallel hooks are out of order by at most one second, which a window of minutes cannot turn into a wrong count worth fixing.
- code-tests finding 4, the `fileEditSignal` part (no row feeds the parser a PostToolUse line without a file path): no writer produces such a line, so the guard has no reachable behavior to pin. The `RecordEdit` failure half is kept as F7.
- code-tests finding 5 (a registered engine without the capability): the path differs from the unregistered-name row only inside the type assertion of `shuttleengines.EditRecorder`. It yields the same report and exit code, which `TestHookEditVerb` already pins.
- Handler check, `PATTERN-child-processes` and `selfexec.LymBinary` for a hook command: `editRecordHookCommand` names bare `lym` from PATH, as the batching hook (`batchingnudge.go:96`) and both engines' creel hooks (`settings.go:105`, `codexengine/command.go:182`) do. A PATH name never re-execs the running test binary, which is what the rule guards, so this follows the established hook convention and is no violation.
