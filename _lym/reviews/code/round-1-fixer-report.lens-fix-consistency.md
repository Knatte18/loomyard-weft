Task: fix-consistency lens over the round-1 change set.

No findings.

I found no twin left stale. These sibling texts still agree with the changes:
- `internal/shuttleengine/codexengine/editrecord.go:2` already says "per file a patch touched".
- `internal/shuttleengine/claudeengine/editrecord.go:2` stays "one record", which is right for the Edit and Write tools.
- `internal/shuttleengine/codexengine/doc.go:149` and `internal/shuttleengine/claudeengine/doc.go:306` describe only the hook command, not the record count.
- `docs/overview.md:711` and `docs/overview.md:720` agree with each other.
- The `EditRecorder` doc and the `hook-edit` Long text now agree on "each".

I did not check the `internal/shuttleengine/codexengine/doc.go` edit-hook passage beyond line 149, because the fix did not touch it.

Tool calls:
1. A single `grep` across Go, markdown and yaml files, to find twin wordings of "one record" and the `hook-edit` description. My context held only the files the fixer read, not their siblings.
