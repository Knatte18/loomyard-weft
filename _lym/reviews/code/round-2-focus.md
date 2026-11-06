---
round: 2
focus:
  - "internal/beamcli/dobbydeps.go `waitReminderDeps` builds the SLACK recipient with `agentname.FormatProgram` and the sender with `creelcli.NewScopedProgramClient(worktree, slug, dobbyProgram)`: settle whether a test calls the resulting `WaitReminders.Post` and checks recipient, DOBBY signing in the dobby's slug scope, and a surfaced post error."
  - "internal/dobbyengine `readWaits` skips a run with a batten cancel marker, and `tick` runs the wait check only while no anchor change stands: settle whether a test row fails if either behaviour breaks."
---
