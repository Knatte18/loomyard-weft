# Record Codex edits for the edit-loop watch

`codex-edit-loop-recorder` · Note · bug, codex, shuttle

## Seen (orch, 2026-11-17)

The edit-loop watch from `edit-loop-detector` counts an agent's edits of one file and sends the run's parent a notice when they loop.
Only the claude engine implements `EditRecorder`: its sessions carry a PostToolUse hook on `Edit|Write` that records each edit.
A Codex run records no edits, so it never gets the notice.

## Idea

Give the codex engine an `EditRecorder` and an edit hook that records each file edit as the same slim line the claude engine writes,
so the engine-neutral watch in `internal/shuttleengine/editloop.go` covers Codex runs unchanged.
Codex edits files through its patch tool rather than `Edit` or `Write`,
so the hook reads the file paths from that tool's input; one patch touching several files records one edit per file.
