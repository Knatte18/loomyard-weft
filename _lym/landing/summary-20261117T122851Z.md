# Notify a run's parent when an agent edits one file again and again

An agent that edits the same file over and over without progress used to run until its time budget ran out.
Shuttle now counts each agent's edits of each file in an autonomous run.
When one file's edits reach a set count within a set window, it sends the run's parent one notice.
The session keeps running; the notice stops nothing and changes no outcome.

## What changes

- **Edit hook (claude engine).**
  For a run that is not interactive, while the watch is on, a Claude session's settings carry a PostToolUse entry matching `Edit|Write`.
  It runs the new hidden verb `lym shuttle hook-edit claude <events file>; true`.
  `Claude.RecordEdit` appends a stamp line and a slim PostToolUse line to `events.jsonl`.
  The line holds only the session id, the agent id, the tool name and `tool_input.file_path`.
  Claude Code's full payload carries the edit's text and the whole file's content.
  Recording it raw would bloat a file that every watch of the run rereads.
- **New signal.**
  `ParseSessionSignals` reads that line as the new `SessionSignalFileEdit`, carrying `FilePath` and `AgentID`.
  `AgentID` is empty for the session's own edit and set for a subagent's or fork's, so each agent counts apart.
- **Watch and notice (engine-neutral).**
  `internal/shuttleengine/editloop.go` adds `watchEditLoops`, run each wait tick beside the dialog watch.
  It counts edits per agent and file at the hook-side stamp time.
  When `edit_loop_edits` of them fall within `edit_loop_window_s`, it sends one notice line through the runner's notifier and restarts that count.
  This is the same channel the hold and dialog notices use.
  The line names the strand, the file, the subagent when one made the edits, the count and the window.
  The agent-written parts are delimited, as in the hold notice.
  `RunState.EditLoopNotifiedOffset` is persisted before the notifier call, so a reattach never repeats a notice.
  Interactive runs, runners with no notifier, and runs under a pause send nothing.
  Under a pause no edits are read, so they count once it lifts.
- **Config.**
  `shuttle.yaml` gains `edit_loop_edits: 4` and `edit_loop_window_s: 300`, the survey's four edits in five minutes.
  `0` edits turns the watch off and installs no hook.
  A negative count, or a non-positive window while the count is on, is refused at load, naming its way forward.
- **Session fold.**
  `SessionFold` skips file edits, so an edit between a turn start and an idle notice does not hide the contradiction that pair signals.

## Design decisions

- The board idea says shuttle already gets every PostToolUse event.
  It gets them as hooks only: `events.jsonl` records no PostToolUse line.
  So this change adds a dedicated slim recording hook instead of reading an existing record.
- The counting lives in the engine-neutral wait loop, not in the hook process.
  The window then runs on recorded stamp times, the run clock covers unstamped lines, and the notice reuses the notifier.
  The hook stays a stateless append.
- "Raises a friction note to the run's parent" is implemented as a shuttle notice line to the parent,
  the parent channel shuttle owns.
  No file is written to the run's friction directory: shuttle is never told that directory,
  and `PATTERN-run-records` keeps Go-written friction notes to their existing writers.
- Only the claude engine implements `EditRecorder`.
  The codex engine has no edit hook yet, so its runs get no edit-loop notice.

## Tests

- `TestWait_EditLoopNotice` covers the watch through `Wait`:
  the notice and the persisted offset, a subagent's loop, session and subagent counting apart, files counting apart,
  the window dropping an old edit, the count restarting after a notice, a pause, a reattach,
  and the off, interactive and no-notifier cases.
- `TestRecordEdit` and `TestBuildSettings_EditHook` pin the slim record read back as a signal, and the hook's settings entry.
- `TestHookEditVerb` and `TestHookEditVerb_RunsOutsideAHub` pin the verb.
  `TestLoadConfig_EditLoop` pins the config, and a new `TestSessionFold` row pins the fold.
