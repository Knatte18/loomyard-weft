1. LOW: `internal/fabricengine/doc.go:416-420`, F3 left consumer-read narrative in place.
   - Evidence: "Both call for the same staleness check." and "It calls `f.code.ChangedFilesSince(sha)` only if the SHA exists" describe a read no production caller makes.

2. LOW: `internal/fabricengine/doc.go:487-491`, the same stale premise.
   - Evidence: "A consumer checking staleness in a tight loop" and "A polling consumer should check `ChangedFilesSince` first" describe consumers that do not exist in production.

F1 proposal: no breakage found; `shedcheckkit -> shedbuild` adds no import cycle and breaks no import rule.

Tool calls:
1. `grep` over `internal/fabricengine/doc.go` for leftover consumer or staleness wording, because the context held only the pre-edit text of the moved lines.
2. `grep` for `shedbuild` in the `shedbuild` and `internal/testkit` enforcement tests, to see whether the new kit import breaks a rule.
