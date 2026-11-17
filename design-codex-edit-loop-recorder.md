# Record Codex edits for the edit-loop watch

`codex-edit-loop-recorder` · Task · bug, codex, shuttle · done

## Seen (orch, 2026-11-17)

The edit-loop watch from `edit-loop-detector` counts an agent's edits of one file and sends the run's parent a notice when they loop.
Only the claude engine implements `EditRecorder`: its sessions carry a PostToolUse hook on `Edit|Write` that records each edit.
A Codex run records no edits, so it never gets the notice.

## Idea

Give the codex engine an `EditRecorder` and an edit hook that records each file edit as the same slim line the claude engine writes,
so the engine-neutral watch in `internal/shuttleengine/editloop.go` covers Codex runs unchanged.
Codex edits files through its patch tool rather than `Edit` or `Write`,
so the hook reads the file paths from that tool's input; one patch touching several files records one edit per file.

## Built since (tidy, 2026-11-17)

Commit `b2cd8150` (`codex-question-deny-nudge`) gave Codex launches `hooks.PreToolUse` and `hooks.PostToolUse` settings, with a `*`-matcher `PostToolUse` hook running `lym shuttle hook-batching codex`.
The hook channel this note needs now exists on the Codex launch line, and that nudge already reads `apply_patch` calls from the rollout at `transcript_path`.
The edit recorder itself is still not built.

Since `1b405351` the watch counts an edit only when it repeats an earlier one, by a `content_hash` on each edit record, so the Codex hook also records the SHA-256 of each edited file's content, read right after the patch, as the claude hook does (tidy, 2026-11-17).
