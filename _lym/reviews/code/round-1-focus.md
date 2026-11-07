---
round: 1
focus:
  - "internal/shuttleengine/claudeengine settings.go and scope.go: does the per-session document keep only per-session or per-role settings, with attribution and promptSuggestionEnabled gone, and are hooks naming session paths and role-bound settings kept?"
  - "claudeengine scope.go enabledPlugins and permissions.deny: is each per-role difference removed only where the same-tool-set rule requires it, and does a setting that becomes uniform move to heddle's shipped.json?"
  - "internal/heddleengine agentdefaults.go and internal/agentconfig: are the model and effort defaults rendered into the claude-settings entry from agents.yaml as the single source, with per-role effort left on the launch in command.go?"
  - "internal/heddleengine overview.go, render.go, replayer.go: does every heddle write render HEDDLE.md in Bolt beside heddle.json, covering shipped and hub entries grouped by target with name, targets, origin and text, a fenced json block for claude-settings object entries, rewritten whole with no extra state?"
  - "Failure paths in the HEDDLE.md render and the agents.yaml read: a missing or invalid agents.yaml, a write error on HEDDLE.md, and whether the replayer stays deterministic and idempotent."
  - "Tests: overview_test.go, replayer_test.go, settings_test.go and the claudeengine golden file for duplicated coverage or assertions on static prose, and for new behavior left uncovered."
  - "PATTERN-heddle-instructions.md, docs/overview.md and the heddleengine and claudeengine doc.go files: do they land in the same diff and match the code, and does the change description or any commit message claim something the diff does not bear out?"
---

Seed focus only; the reviewer still reads the whole committed range.
