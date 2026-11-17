---
round: 1
focus:
  - "internal/shuttleengine/codexengine/editrecord.go: does the apply_patch input parse cover every file operation (add, update, delete, move), and does one patch touching several files record one edit per file?"
  - "internal/shuttleengine/codexengine/editrecord.go: is the content hash read after the patch, and what is recorded for a deleted or unreadable file? Does it match the claude hook's behavior the edit-loop watch relies on?"
  - "internal/shuttleengine/codexengine/editrecord.go and the claude engine's editrecord.go: is the recorded line identical in shape to the claude engine's slim line, and does the refactor to shuttleengine.FileContentHash leave the claude hook's behavior unchanged?"
  - "internal/shuttleengine/codexengine/command.go and codexengine.go: how does the new apply_patch PostToolUse hook coexist with the existing '*'-matcher hook-batching hook; is the hook ordering, matcher and command string right, and does a hook failure stay non-fatal for the Codex run?"
  - "internal/shuttleengine/codexengine/events.go: does ParseSessionSignals read the recorded lines as file edits without double counting edits that the rollout's own apply_patch calls already produce?"
  - "internal/shuttleengine/editloop.go: what does the 21-line addition do, and is it correctly engine-neutral for the claude engine's existing behavior?"
  - "Scope against the Goal in the decision record: is each change licensed (EditRecorder on codex, hook, content_hash, shared hash helper), and does the doc.go update land with the change?"
  - "Tests in editrecord_test.go, prepare_test.go, command_test.go, new_test.go: is each new behavior covered once, and does any test only restate static hook setting strings already pinned elsewhere?"
---

Round 1 focus on the diff `96b20f1c..HEAD`: the Codex edit recorder, its hook wiring, and the shared hash helper.
