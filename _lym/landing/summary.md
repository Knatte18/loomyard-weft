# Record Codex edits for the edit-loop watch

Codex runs now feed shuttle's edit-loop watch, so a Codex agent that undoes or repeats its own edits notifies its parent as a claude agent already did.
Before this change only the claude engine implemented `EditRecorder`, so a Codex run recorded no edits and never got the notice.

## What changes

- `codexengine` gets an `EditRecorder`.
  While `shuttle.yaml`'s `edit_loop_edits` is above 0, an autonomous (non-interactive) Codex launch carries an `apply_patch`-matcher `PostToolUse` hook, placed after the activity hook.
  It runs `lym shuttle hook-edit codex <edit records file>` and prints the empty hook output, so a missing or failing `lym` shows the session no hook error.
- `RecordEdit` reads the file headers of the patch text in `tool_input.command` and appends one record per touched file.
  The file is the one an add, update or delete header names, or the target of a move header after an update.
  A file named twice in one patch is recorded once, and a relative path resolves against the payload's `cwd`.
  A relative path with no `cwd` is skipped.
- Each record is the same slim line the claude engine writes, with lym's `content_hash` of the file as the hook reads it after the patch.
  A file that cannot be read, such as a deleted one, is recorded without a hash, and an edit with no hash never counts toward a loop.
- A payload for another tool, one naming no file, or one whose `tool_response` reports a failed patch appends nothing.
- `ParseSessionSignals` reads such a line as a `SessionSignalFileEdit` with its file path, agent id and content hash.
  The engine-neutral watch in `internal/shuttleengine/editloop.go` then covers Codex unchanged.
- `docs/overview.md`, the `codexengine` package doc, and the `hook-edit` help and `EditRecorder` comments now describe a record per edited file and name the Codex hook.

## Design decisions

- **One record per file.**
  One patch can touch several files, and the watch counts edits per file, so the hook writes a stamp line and a record for each file in a single write.
- **Agent id from the rollout name.**
  A Codex payload carries no agent id, and a child thread's payload carries its parent's session id.
  The hook takes the agent id from the thread id its `transcript_path` rollout name ends in, and leaves it empty when that name ends in the session id.
- **Failed patches are not recorded.**
  `tool_response` has no fixed shape in Codex, so `patchFailed` reads only a failure marker in an object: `success` false, or a non-empty string or object `error`.
  Any other response, including an absent or non-object one, reads as a success.
- **Shared hash helper.**
  The SHA-256 helper moved from `claudeengine` to `shuttleengine.FileContentHash`, so both engines hash a file the same way.

## To check by hand

- No run against a real Codex session was done; the hook's input shape rests on the `apply_patch` payload the batching nudge already reads.
  Check that a real `apply_patch` call produces the records, and that the `tool_response` of a failed patch is recognised by `patchFailed` in `internal/shuttleengine/codexengine/editrecord.go`.
- `record-edit-failed-apply-patch` (design, medium): a failed `apply_patch` could be recorded as an edit.
  `patchFailed` covers the shapes it knows; check it against the real failure output.
- `overview-codex-edit-hook-missing` (consistency, medium): `docs/overview.md` omitted the Codex edit hook.
  The diff adds it to the shuttle paragraph; check that the wording matches the other Codex hook entries.
