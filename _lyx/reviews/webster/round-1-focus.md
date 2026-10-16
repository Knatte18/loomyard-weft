---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/command.go: does `buildOpenCmd` add the skip flag on both the fresh and the resume line, and does `OpenLaunch` validate the mode with the existing interactive rule rather than a new one?"
  - "internal/shuttleengine/open.go and engine.go: is `PermissionMode` carried blind to `OpenLaunch` with no validation outside `claudeengine`, and is it recorded on the opened-session log line?"
  - "internal/shuttlecli/open.go: does `--permission-mode` fill `OpenSpec.PermissionMode` and does its help text name `bypass`, `prompt` and the empty default as the plan's card 3 states?"
  - "internal/shuttleengine/claudeengine/command_test.go, internal/shuttlecli/cli_test.go and open_test.go: do the new tests duplicate existing coverage, and is each added behavior covered (test economy)?"
---

Seed focus derived from the plan's three cards and the diff's file list; no prior round exists.
