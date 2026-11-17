```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke, and file a finding for each behaviour with no such row.

**Per-behaviour list**

| Behaviour | Pinned by |
|---|---|
| The `hook-edit` verb appends the record, prints nothing and exits 0 on an unknown engine or malformed input | `TestHookEditVerb` (the three rows) |
| The `hook-edit` verb skips hub resolution | `TestHookEditVerb_RunsOutsideAHub` |
| `hook-edit` with a registered engine that has no `EditRecorder` (`shuttleengines.EditRecorder` returns false for a known name) | none (only an unregistered name is exercised) |
| `RecordEdit` writes a slim stamp-and-record pair, with the session id and the agent id, and leaves out the payload's text | `TestRecordEdit` (the `AnEditOfTheSessionIsAFileEdit` and `ASubagentsWriteNamesTheAgent` rows) |
| `RecordEdit` appends nothing for another tool or a missing file path, and errors on a malformed payload | `TestRecordEdit` (the remaining rows) |
| `RecordEdit` returns an error when the events file cannot be opened or written | none |
| `fileEditSignal` gives no signal for a PostToolUse line that names no file | none (no row feeds the parser such a line) |
| The settings carry the hook only when the watch is on and the run is not interactive | `TestBuildSettings_EditHook` |
| The config: template values, 0 turns the watch off, a negative count and a non-positive window while on are refused with the way forward | `TestLoadConfig_EditLoop` |
| `SessionFold` skips file edits | `TestSessionFold` (the new row) |
| `fold` counts per agent, per file, inside the window, and restarts the count after a notice | `TestWait_EditLoopNotice` (the subagent, apart, window-drop, across-ticks and six-edits rows) |
| `fold` uses the signal's stamp time, not the run clock, when `At` is set | none |
| The notice carries the strand, file, subagent, count and window, with the agent text delimited | none independent: the expected value is built by `editLoopNotice` itself |
| The offset is persisted before the notifier call; a reattach does not repeat a notice | `TestWait_EditLoopNotice` (`wantPersisted`, the reattach row) |
| The pause, interactive, no-notifier and off guards | `TestWait_EditLoopNotice` (the matching rows) |
| A failed persist skips the batch's notices; a notifier error is only logged | none |
| A notice changes no outcome | `TestWait_EditLoopNotice` (the `OutcomeTimeout` assertion) |

**Findings**

1. **MEDIUM, design.** Location: `internal/shuttleengine/editloop_test.go:801` (`TestWait_EditLoopNotice`) against `internal/shuttleengine/editloop.go` `fold`.
   No test pins that an edit is counted at its hook-side stamp time.
   The test builds `&sessionFakeEngine{}` without `markEvents` (`editloop_test.go:876`).
   The fake sets `signal.At` only under `markEvents` (`fakes_test.go`, the `if e.markEvents { signal.At = stamp }` branch).
   So every row's edits have a zero `At` and take `run.clock.Now()`.
   A regression that ignored `signal.At` would pass every row, including the pause row, which the summary presents as relying on stamp times.
   Needed: a row with stamped edits whose stamps span more than the window but are read in one batch, wanting no notice, plus the inverse.

2. **LOW, design.** Location: `internal/shuttleengine/editloop_test.go:806-808`.
   The expected notice is `editLoopNotice(...)` itself, so no row pins what the line carries.
   Dropping the file, the subagent id, the count or the window, or removing the delimiters, still passes.
   Only the `Shuttle notice:` prefix and the single-line shape are checked independently (`editloop_test.go:927-931`).
   The filled values (file path, agent id, count, window) are Go-filled values, not static prose, so a row asserting them is allowed.

3. **LOW, design.** Location: `internal/shuttleengine/editloop.go`, `watchEditLoops` failure paths.
   Two paths have no row: a failed `persistEditLoopNotifiedOffset` skips the batch's notices and leaves the in-memory cursor advanced, and a notifier error is only logged.
   No row injects a failing state save or a failing notifier.

4. **LOW, design.** Location: `internal/shuttleengine/claudeengine/signals.go` `fileEditSignal` and `internal/shuttleengine/claudeengine/editrecord.go` `RecordEdit`.
   Neither function's negative path is pinned.
   For `fileEditSignal`: no `TestRecordEdit` row feeds the parser a PostToolUse line with no `file_path`. The `NoFilePathAppendsNothing` row stops at `RecordEdit` and the parser reads an empty file.
   For `RecordEdit`: no row covers an open or write failure being returned as an error.

5. **NIT, design.** Location: `internal/shuttlecli/hookedit_test.go:157`.
   No row covers a registered engine without the capability (codex).
   That path differs from an unregistered name inside `shuttleengines.EditRecorder` (`shuttleengines.go:316`), yet both report the same message.

No test asserts on static stencil or spec prose. The `lym config shuttle` check in `TestLoadConfig_EditLoop` pins a refusal's way forward, which `PATTERN-refusal-way-forward` asks for.

Tool calls:
1. `sed` of `internal/shuttleengine/fakes_test.go:900-945` plus a grep for `dialogTick`: the manifest held only the fake's EDIT hunk, not how `At` is set (only under `markEvents`) or the tick length.
