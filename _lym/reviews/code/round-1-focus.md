---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/settings.go: does every document the package composes (the run document in a hub and standalone, and the opened session's creel-only document) emit `attribution` as `{\"commit\": \"\", \"pr\": \"\"}`, and is there any other settings writer or code path in the package or its callers that composes a Claude settings document and the diff leaves without it?"
  - "Scope against the decision record's `## Goal` Wanted list: does the diff change anything beyond the settings documents, the package doc and `docs/overview.md`, and does it leave the landing's credit block untouched?"
  - "docs/overview.md and internal/shuttleengine/claudeengine/doc.go: does each state the attribution beside the other settings every document carries, as the Wanted list asks? Check that the overview change actually names it, not only that the diff touches the file."
  - "internal/shuttleengine/claudeengine/command_test.go and testdata/settings-golden.json: do the edits only follow the new key, or do they weaken an existing assertion on hooks or the open-launch settings?"
  - "internal/shuttleengine/claudeengine/settings_test.go `TestSettings_CreditNoSession`: judge it under Test economy against the golden file and the command test, which now also carry the key. Name the covering test if it duplicates one."
  - "Commit messages: check that the trailer and body claims match the diff, and that no commit credits an engine in a way the change itself forbids."
---

Small diff in one package; the decision record carries only a Goal, so the Wanted list is the measuring stick.
