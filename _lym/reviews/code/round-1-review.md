---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/editrecord.go#RecordEdit
    summary: RecordEdit records every file a patch names without checking the apply_patch call succeeded, so retries of a failing patch each record the unchanged file's hash and the watch counts them as repeats
    fix: In internal/shuttleengine/codexengine/editrecord.go, add the payload's tool_response to editHookInput and return nil without appending when it reports a failed or rejected apply_patch call. Add a TestRecordEdit row whose payload carries a failed tool_response and wants no records. If Codex is confirmed to fire no PostToolUse hook for a failed apply_patch, leave the code as is and say so in RecordEdit's doc comment instead.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: consistency
    location: docs/overview.md:711
    summary: The overview still says only an autonomous claude run's Edit and Write calls run lym shuttle hook-edit, and its Codex paragraph lists the batching and creel PostToolUse hooks but not the new apply_patch edit hook
    fix: In docs/overview.md, change line 711 so it says an autonomous run's file edits, Edit and Write on claude and apply_patch on codex, run the hidden `lym shuttle hook-edit`. In the Codex paragraph that lists the `*`-matcher PostToolUse batching hook, add that while `edit_loop_edits` is above 0 an autonomous Codex launch also carries an `apply_patch`-matcher PostToolUse hook running `lym shuttle hook-edit codex`, which records each file a patch touches.
    origin: lens:code-affected-set
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/editrecord.go#patchedFiles
    summary: patchedFiles matches the file headers on the line with only a trailing carriage return removed, so a header carrying leading whitespace, which Codex's patch parser accepts, records no edit
    fix: In patchedFiles, replace `line = strings.TrimRight(line, "\r")` with `line = strings.TrimSpace(line)`, so a header with leading or trailing whitespace is matched. Add a header with leading spaces to one TestRecordEdit row's patch.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/editrecord_test.go#TestRecordEdit
    summary: Every TestRecordEdit row calls RecordEdit once on a fresh records file, so no test fails if a second call truncates the records an earlier call appended
    fix: In TestRecordEdit, give one row a second patch, or add a row, that calls RecordEdit twice on the same records path, changing the edited file between the calls. Assert that ParseSessionSignals reads both edits back in order, each with its own content hash.
    origin: assist:assist
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/editrecord.go#childThreadID
    summary: childThreadID's branches for an empty session id, an empty transcript_path, and a rollout name of threadIDLength characters or fewer have no test row
    fix: Add TestRecordEdit rows, or a table test of childThreadID, that send a payload with no session_id and a transcript_path whose base name is a bare 36-character thread id. Assert the agent id each one records.
    origin: lens:code-tests
  - id: F6
    severity: NIT
    class: design
    location: internal/shuttleengine/codexengine/events.go#fileEditSignal
    summary: No test feeds ParseSessionSignals a PostToolUse line without tool_input.file_path, so dropping fileEditSignal's empty-path guard would leave every test green
    fix: Add a row to the existing ParseSessionSignals table test in internal/shuttleengine/codexengine/events_test.go. Give it a stamp line and a PostToolUse line with no tool_input.file_path, and want no signal.
    origin: lens:code-tests
  - id: F7
    severity: NIT
    class: consistency
    location: internal/shuttlecli/hookedit.go:27
    summary: The hook-edit verb's Long text says it appends one record of the edited file, but a Codex payload now appends one record per file the patch touches
    fix: In internal/shuttlecli/hookedit.go's Long text, change "appends one record of the edited file and a hash of its content" to "appends a record of each edited file and a hash of its content". In internal/shuttleengine/engine.go, change the RecordEdit doc line "appends the file edit the hook input payload describes" to "appends each file edit the hook input payload describes".
    origin: lens:code-affected-set
---

### [MEDIUM] A failed apply_patch is recorded as an edit

**Location:** internal/shuttleengine/codexengine/editrecord.go#RecordEdit

**Issue:** `editHookInput` reads no outcome field, and `patchedFiles` works from the patch text alone.
A Codex patch that fails to apply leaves the file at the content an earlier edit left, and the hook records that same hash again.
`fold` in `internal/shuttleengine/editloop.go:106` counts every such record as a repeat, so a run of failed patch retries on one file can send the parent a false loop notice.
Failed patch retries are common on Codex.
The claude hook does not have this problem because Claude Code fires PostToolUse only after a successful Edit or Write.
No source in the repository shows whether Codex 0.164.0 fires PostToolUse for a failed `apply_patch`, so the fix asks for that check either way.

### [MEDIUM] The overview still says only claude runs record edits

**Location:** docs/overview.md:711

**Issue:** Line 711 names only an autonomous claude run's `Edit` and `Write` calls as running `lym shuttle hook-edit`.
The Codex paragraph that follows lists the Codex `PostToolUse` hooks, batching and creel, without the new edit hook.
So the user-facing doc still says Codex runs get no edit-loop notice, which is the opposite of what the diff makes true.

### [LOW] A patch header with leading whitespace is missed

**Location:** internal/shuttleengine/codexengine/editrecord.go#patchedFiles

**Issue:** The `HasPrefix` checks run on `strings.TrimRight(line, "\r")`.
Codex's own patch parser trims a header line before matching it, so a patch whose header is indented applies but records nothing.
That case is rare, and `TrimSpace` covers it at no cost.

### [LOW] Appending across RecordEdit calls is untested

**Location:** internal/shuttleengine/codexengine/editrecord_test.go#TestRecordEdit

**Issue:** Each row builds a fresh records path and calls `RecordEdit` once.
If `os.O_APPEND` were replaced by `os.O_TRUNC`, every row would still pass, but the watch would never see more than the latest patch's edits.

### [LOW] childThreadID's fallback branches are untested

**Location:** internal/shuttleengine/codexengine/editrecord.go#childThreadID

**Issue:** Every row sends a non-empty session id and a rollout name longer than 36 characters.
The early return for an empty session id or transcript and the whole-name return for a short name could change without any test failing.

### [NIT] fileEditSignal's empty-path guard is untested

**Location:** internal/shuttleengine/codexengine/events.go#fileEditSignal

**Issue:** `RecordEdit` never writes an empty path, and `events_test.go` has no PostToolUse row.
So nothing pins that a PostToolUse line naming no file yields no signal.

### [NIT] Hook docs still describe one record per hook input

**Location:** internal/shuttlecli/hookedit.go:27

**Issue:** The verb's Long text and the `EditRecorder.RecordEdit` doc in `internal/shuttleengine/engine.go:322` describe one record per payload.
A Codex `apply_patch` payload now yields one record per file.

## Holistic notes

The diff builds every goal point of the board entry.
The codex engine has an `EditRecorder`.
An `apply_patch` PostToolUse hook writes the claude engine's slim line shape with `content_hash` read after the patch.
One patch records one edit per file.
`ParseSessionSignals` reads the records back, so the engine-neutral watch reaches Codex runs without changing.
The hash helper moved verbatim, so the claude hook behaves as before.
The hook sits after the activity hook and before the batching hook in the one PostToolUse setting.
It ends in the empty hook output, so a lym failure never reaches the session.
The watch reads the separate edit records file, and the events file carries no PostToolUse payload, so no edit is counted twice.
No PATTERN entry is violated, and none was declared to change.

## Rejected

- lens:code-affected-set LOW, the second `contentHashKey`/`editRecord` against PATTERN-code-conventions: each engine's record is written and read back inside that engine's own package, by its `RecordEdit` and its `ParseSessionSignals`. No cross-package reader depends on the key, so there is no shared name that could go stale. Per-engine payload shapes stay under the engine's own package, as PATTERN-shuttle-provider-seam asks.
