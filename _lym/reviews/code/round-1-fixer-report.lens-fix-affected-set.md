No findings. Each finding whose premise is a set is covered.

- **F2 (docs naming the edit hook):** `docs/overview.md:711` and `:720` carry the update. `codexengine/doc.go:149-160` already describes the Codex hook. The other `hook-edit` mentions are claude-only or engine-neutral.
- **F7 (docs saying one record per payload):** `hookedit.go` and `engine.go` are fixed. The `EditHookVerb` doc at `engine.go:310` says "the edited file", which is neutral. `claudeengine/doc.go:306` is claude-only, where one record per edit is correct. No other site states one record per payload.
- **F1 (sites that read a tool outcome):** `RecordEdit` is the only Codex site that reads one.
- **Left unchecked:** the F3, F4, F5 and F6 test-gap findings. Each names a single function, so there is no set to rerun.

Tool calls made:
1. `grep` for the hook-edit, edit-loop and record-per-payload doc wording across `.go`, `.md` and `.yaml`. My context lacked a repo-wide view of the doc sites.
2. `grep` for `editRecordHookCommand` and `EditHookVerb`. My context lacked their callers.
