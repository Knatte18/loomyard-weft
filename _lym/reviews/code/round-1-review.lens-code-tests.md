```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I named the test row that would fail if it broke.

**Behaviour → test that pins it**
- The claude hook still records `content_hash` through `shuttleengine.FileContentHash`: `claudeengine/editrecord_test.go:47` `AnEditOfAReadableFileCarriesItsContentHash`.
- Codex `FileContentHash` for a readable file: `TestRecordEdit/AnUpdateOfAReadableFileCarriesItsContentHash`. For an unreadable or deleted file it records no hash: `TestRecordEdit/EachFileOfAPatchIsRecordedOnce` (the `gone.go` row has no hash).
- `patchedFiles`:
  - add, update and delete headers, CRLF, a file named twice recorded once, patch order: `EachFileOfAPatchIsRecordedOnce`.
  - a move records its destination: `AMoveRecordsTheFileItMovesTo`.
  - a relative path with no cwd is skipped: `ARelativePathWithNoCwdAppendsNothing`.
  - a patch naming no file: `APatchNamingNoFileAppendsNothing`.
  - an absolute path that is written successfully: none. The only absolute row, `AnUnopenableRecordsFileIsAnError`, fails before any record is checked. `resolvePath` is shared with, and covered by, the batching nudge's tests, so I raise no finding.
- `RecordEdit`:
  - another tool: `AnotherToolAppendsNothing`.
  - a malformed payload: `AMalformedPayloadIsAnError`.
  - an open failure: `AnUnopenableRecordsFileIsAnError`.
  - a stamp per file and every record consumed: each row's `consumed` and `At` checks.
  - no patch text or file content in the records: each row's `***`/`package` check.
- `childThreadID`:
  - the session's own rollout gives an empty agent id: the default `mainRollout` rows.
  - a child rollout gives its trailing 36 characters: `AChildThreadsPatchNamesItsThread`.
  - an empty session id, or a rollout name of 36 characters or fewer: none.
- `fileEditSignal`:
  - a PostToolUse line becomes a file edit with agent id and hash: every `TestRecordEdit` row reads its records back through `ParseSessionSignals`.
  - a PostToolUse line with no `file_path` yields no signal: none.
- `hookSettings`:
  - the edit hook on `apply_patch` sits after the activity hook and before the batching hook: `TestHookSettings/EditHookBeforeTheBatchingHook`.
  - the literal command ending in `; printf '{}\n'`, so a failure is non-fatal: the `editRecordHookCommand` assertion in `TestHookSettings`.
- `Prepare` gate (`EditLoopEdits > 0 && !Interactive`):
  - `TestPrepare_SwitchedToolHooks` rows `Autonomous`, `BatchingOff`, `Interactive` and `EditLoopOff`.
  - The whole PostToolUse setting string must appear in the launch line, so a hook that should be absent also fails the row.
- `editloop.go` is unchanged apart from the moved helper. `fold` and `watch` need no new row.

**Findings**
1. LOW, class `design`, `internal/shuttleengine/codexengine/editrecord.go:349-361` (`childThreadID`). The branches for an empty session id, an empty `transcript_path`, and a rollout name of `threadIDLength` characters or fewer (which returns the whole name) have no row. Changing them would leave `TestRecordEdit` green. Evidence: the rows at `editrecord_test.go:428-458` always send a non-empty `sessionID` and a rollout name longer than 36 characters.
2. NIT, class `design`, `internal/shuttleengine/codexengine/events.go:155-157` (`fileEditSignal`). No test feeds a PostToolUse line without `tool_input.file_path`, so dropping the empty-path guard would emit a file-edit signal with an empty path and no test would fail. Evidence: `events_test.go` has no PostToolUse or FileEdit row (grep came back empty), and `RecordEdit` never writes an empty path.

Tool calls:
1. grep of `claudeengine/editrecord_test.go`, `codexengine/events_test.go` and the codexengine tests for the hash, PostToolUse parsing, `FileContentHash` and child-thread coverage. My context held the diff's tests but not these existing tests.
