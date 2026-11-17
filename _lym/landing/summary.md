# Record Codex edits for the edit-loop watch

Only the claude engine implemented `shuttleengine.EditRecorder`, so a Codex run recorded no file edits.
The edit-loop watch never saw one and never sent the parent a notice.
This change gives the codex engine an edit hook that writes the same slim records,
so the engine-neutral watch in `internal/shuttleengine/editloop.go` covers Codex runs unchanged.

## What changes

- **Edit hook on the launch line.**
  While shuttle.yaml's `edit_loop_edits` is above 0 and the spec is not interactive, `Prepare` adds a `PostToolUse` entry matching `apply_patch`, after the activity hook.
  The entry runs `lym shuttle hook-edit codex <edit records file>` and then prints the empty hook output.
  The records file is `shuttleengine.EditRecordsFileName` in the run directory, the file the watch reads.
- **`Codex.RecordEdit`** (`internal/shuttleengine/codexengine/editrecord.go`).
  Codex edits files through its patch tool, so the hook reads the file headers of the patch text in `tool_input.command`: add, update, delete, and a move's destination.
  One patch touching several files records one edit per file, and a file the patch names twice is recorded once.
  A relative path is resolved against the payload's `cwd`.
  Each record is a stamp line plus the slim `PostToolUse` line the claude hook writes.
  That line holds the session id, the agent id, the tool name, `tool_input.file_path` and `content_hash`, the SHA-256 of the file read right after the patch.
  A deleted or unreadable file gets no hash, so that edit never counts toward a loop, as on Claude.
- **Child threads.**
  A Codex payload carries no agent id, and a child thread's payload carries its parent's session id.
  The hook reads the child's id from the name of its rollout file (`transcript_path`), which ends in the thread id.
  This keeps a parent's and a child's edits of one file from counting as one agent's repeats.
- **Signals.** Codex's `ParseSessionSignals` now reads a recorded `PostToolUse` line as a `SessionSignalFileEdit` with its file, agent id and hash.
  The events file never carries `PostToolUse` payloads, since the activity hook only touches a stamp, so nothing else changes how it reads.
- **Shared hash.** The content hash moves from claudeengine into `shuttleengine.FileContentHash`, used by both engines' hooks.

## Design decisions

- The hook matches `apply_patch` alone, so no other tool call starts a lym process.
  This mirrors the claude hook's `Edit|Write` matcher.
- The launch manifest gets no edit-hook item, because the claude engine's manifest lists none either, and the parity comparison stays balanced.
- The payload's patch text is read from `tool_input.command`, the field the batching nudge's tests already use for `apply_patch` payloads.

## Tests

- `TestRecordEdit` (codexengine) pins the records written for a hashed update and for a multi-file patch with a CRLF body, a repeated file and a delete.
  It also covers a move, a child thread's id, a relative path with no cwd, a patch naming no file and another tool.
  It also covers a malformed payload and an unopenable records file, and every row reads its records back through `ParseSessionSignals`.
- `TestHookSettings` adds the edit hook's place in the `PostToolUse` setting and its literal command.
- `TestPrepare_SwitchedToolHooks` (formerly `TestPrepare_QuestionDenyAndBatchingHook`) adds rows for the edit hook.
  An autonomous spec carries it, and an interactive spec or a watch that is off does not.
