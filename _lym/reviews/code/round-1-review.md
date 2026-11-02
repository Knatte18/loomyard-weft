---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttlecli/hookbatching.go:20
    summary: the hidden hook-batching verb has no test, so its exit-0-on-every-path promise and its output are unpinned
    fix: Add a tier0 test in internal/shuttlecli that runs hookBatchingCmd through cobra with fixture stdin. Rows - an unknown engine name (stderr message, empty stdout, nil error); a state file and transcript fixture that fire a nudge (stdout is the hook output plus a newline); no breach (empty stdout); a malformed payload (error to stderr, nil from RunE). Reuse the transcript fixture style of batchingnudge_test.go through a small helper in shuttlecli's own test file.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:31
    summary: writeNudgeTranscript puts every call of a message on one line, so merging the split lines of one message id, and skipping sidechain and id-less lines, in tailMessages is unpinned
    fix: In batchingnudge_test.go extend writeNudgeTranscript, or add a second fixture helper, so one message can span several assistant lines (a text block line, then one line per tool_use, all sharing message.id). Add TestBatchingNudge rows that (a) fire the lookup chain when the current message's calls are split across lines yet it still counts as one message, (b) stay silent when a sidechain assistant line or an assistant line with no message id sits between single-lookup messages.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/batchingnudge.go:209
    summary: the transcript tail window (cut first line dropped, window with no newline, file smaller than the budget) has no test
    fix: Make the budget a parameter of BatchingNudge's tail read or expose readTranscriptTail through export_test.go, then add a table test over readTranscriptTail with budget values that cover a file under the budget (read whole), a file over it (first line dropped, only complete lines returned) and a window holding no newline (nil, nil).
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:104
    summary: TestBatchingNudge rows miss the boundaries of sliced(), the off rows of read_slice and edit_chain, the breach order, a multi-call edit message, a mixed edit and read message, and the slice_lines edge
    fix: Add rows to the TestBatchingNudge table in batchingnudge_test.go. (a) A Read with limit only fires, with offset 1 only is silent, with neither is silent (a sliced field of offset and limit pointers replaces the bool). (b) read_slice off and edit_chain off, each silent where its breach would fire. (c) A fixture where the lookup chain and a short-file slice both hold, wanting the lookup nudge. (d) An edit message holding two Edit calls counts as one edit message, and an Edit plus Read message is silent. (e) A file of exactly slice_lines lines read with a slice is silent, and a last line without a newline counts.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:204
    summary: failure paths of BatchingNudge and the bounds of ValidateBatchingNudge are pinned by one row each at most
    fix: Add rows to TestBatchingNudge for a malformed payload (error returned), an empty transcript_path or tool_use_id (silent), an unreadable transcript (silent, no error), a tool_use_id absent from the tail (silent), and a corrupt state file (error). Turn the refusal rows of TestPrepare_BatchingNudgeHook into a bounds table covering a negative context_tokens (refused), context_tokens 0 (accepted), edit_run, slice_lines and max_nudges at 0 (refused), and an enabled false block with bad values (accepted).
    origin: lens:code-tests
---

### [MEDIUM] hook-batching verb is untested

**Location:** internal/shuttlecli/hookbatching.go:20
**Issue:** The verb is the process the PostToolUse hook runs, and the diff's own promise is that it exits 0 on every path and prints either the engine's hook output or nothing. `TestBatchingNudger` in `internal/shuttleengines` covers only the registry lookup and the no-state-file exit of the engine. No test runs `hookBatchingCmd`, so an unknown engine, an unreadable stdin, an engine error and the written output line are all unpinned, as is the `PersistentPreRunE` skip that lets the verb run outside a hub.

### [MEDIUM] Transcript fixture never splits a message across lines

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:31
**Issue:** `tailMessages` groups assistant lines by `message.id` because Claude Code writes one line per content block. `writeNudgeTranscript` writes all of a message's calls on one line, so the grouping that makes parallel calls count as one message is exercised only with one line per id, and the skip of sidechain and id-less lines has no row at all.

### [LOW] Tail window of the transcript read is untested

**Location:** internal/shuttleengine/claudeengine/batchingnudge.go:209
**Issue:** `readTranscriptTail` drops the first, possibly cut, line of a window that does not start the file and returns nil for a window with no newline. `batchingTailBytes` is a constant, so no test through `BatchingNudge` can reach either branch without a transcript above 4 MiB.

### [LOW] Boundary rows missing in TestBatchingNudge

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:104
**Issue:** Every sliced row sets offset and limit together, so a limit alone, offset 1 alone and a plain Read are unpinned. Only `lookup_chain` has an off row. No row has two breaches true at once, so the documented order could be swapped unnoticed. No message holds several Edit calls or an Edit with a Read. No file sits at exactly `slice_lines` lines.

### [LOW] Failure paths and validation bounds have one row at most

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:204
**Issue:** The silent and error exits of `BatchingNudge` before the judgment (malformed payload, empty fields, unreadable transcript, call absent from the tail, corrupt state) and the bounds of `ValidateBatchingNudge` (negative and zero `context_tokens`, the other keys, a disabled block) are pinned only by the `lookup_run: 0` refusal and the no-state-file row.

## Rejected

- Lens code-correctness F1 (a parallel message whose later calls are not yet written reads as one single lookup): the Design calls the hook a heuristic that may fire on a right call, and `claudeengine/doc.go` states this bound; the diff's behavior matches the record.
- Lens code-correctness F2 (an earlier sliced Read counts as "already read"): the Design says "a file already read and unchanged since", and two slices of one file over two turns is the pattern the rule forbids, so the nudge fires on a right call; not a defect against the fasit.
- Lens code-correctness F3 (a resumed run may reseed the state file and reset the cap): rejected on evidence, since `launch` in `internal/shuttleengine/run.go:605` calls `createRunDir` before every `Prepare`, so each launch, a resume included, has a new run directory and its own state file.
- Lens code-tests T10 (no fixture row with cache tokens in the usage): `latestReading` is existing code that already sums the three figures, and the threshold comparison is pinned by the below-threshold row; a row for the sum would test `latestReading`, not this diff.
- Lens code-affected-set: no finding to judge.
- Holistic check against the fasit: the three breaches match the Design's definitions in `kindOf`, `runEndingAt` and `judgeBatching`, the config keys and template defaults are all present and validated, the hook installs and seeds state only when enabled, the shipped wording makes each of the five listed changes, and the nudge lines quote that wording; none of this is a finding.
