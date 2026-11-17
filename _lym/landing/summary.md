# Shuttle notices a parent when an agent edits one file again and again

Nothing stopped an agent that kept editing the same file without progress; only the run's time budget ended it.
Shuttle now counts each agent's edits per file and tells the run's parent when one file's edits reach a set count within a set window.
The session keeps running, and the notice is advisory.

## What changes

- A new edit hook runs on each Claude `Edit` and `Write` call.
  It runs the hidden `lym shuttle hook-edit <engine> <edit-records-file>` verb (`internal/shuttlecli/hookedit.go`),
  which appends a slim record of the edited file to a per-run edit records file.
  The record holds the hook-side time, the session and agent ids, and the file path, never the edit's text or the file's content.
  The hook command ends in `; true` and the verb always exits 0, so a failure never shows the session a hook error.
- `internal/shuttleengine/editloop.go` adds the watch to an autonomous run's wait loop.
  It reads new records each tick and counts edits per agent and file; the session and each subagent or fork count separately.
  An edit older than the window before a later edit of the same file by the same agent drops out of the count.
  When the count reaches `edit_loop_edits`, the runner's notifier sends the parent one shuttle notice line,
  the same channel the hold and dialog notices use, and the count for that file restarts.
  The notice delimits the file path and agent id as the agent's own text.
- Two new shuttle config keys: `edit_loop_edits` (default 4) and `edit_loop_window_s` (default 300).
  `edit_loop_edits: 0` turns the watch off and installs no edit hook.
  `LoadConfig` rejects a negative `edit_loop_edits`, and a non-positive `edit_loop_window_s` while `edit_loop_edits` is above 0.
- `RunState.EditLoopNotifiedOffset` persists the records-file offset before the notifier is called,
  so a restarted wait does not re-notify.
  When the state write fails, the watch is restored and the batch is read again on the next tick.
  A notifier error is only logged.
- The watch does nothing for an interactive run, a runner with no notifier, an engine that parses no session signals,
  or while a pause covers the run.
  Edits made during a pause count once it lifts.

## Design decisions

- The friction note is a shuttle notice line through the notifier, not a file in the run's friction directory.
  Shuttle owns the notifier channel to the parent and is never told the friction directory,
  and `PATTERN-run-records` keeps Go-written friction notes to their existing writers.
- The edit hook writes its own edit records file beside the events file, not the events file itself,
  so the watch does not reread the full payloads the events file carries.
- Edit times come from the hook-side stamp, so the window measures when the edits happened, not when the wait read them.
  A record with no stamp takes the read time.
- Only the claude engine has an edit recorder (`shuttleengine.EditRecorder`, registered through `shuttleengines`).
  A codex run gets no notice until a later task adds a codex recorder and hook;
  the engine-neutral watch needs no change then.

## To check by hand

- No manual run was made of the hook inside a live Claude session.
  Check that a real session's `Edit` and `Write` calls produce records, and that four edits of one file within five minutes send one notice.
- The edit hook shares the edit records file with every watch of the run, which is a design choice a reviewer may want to confirm
  (`internal/shuttleengine/claudeengine/editrecord.go`).
- The task's friction note became a notice line, a reinterpretation of the board entry's wording
  (`internal/shuttleengine/editloop.go`, `editLoopNotice`).
- Counting by stamp time is covered by tests, but its edge cases are thin
  (`internal/shuttleengine/editloop_test.go`).

Closes no issue.
