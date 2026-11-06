---
round: 2
focus:
  - "internal/shuttleengine/loadturn.go, loadSkillTurn and settleLoadTurn: for a load turn whose start was read and whose end was not, does the loop keep polling past loadTurnOpenBound until a turn end is read, the pane dies or the run deadline passes, and is a deadline expiry ended as a failed start through run.classifyDeadlineExpiry(OutcomeTimeout) without typing the prompt? A timeout skip should remain only for a turn whose start was never read."
  - "internal/shuttleengine/startup_test.go, TestStartup_LoadTurnAPIError: is there a row that runs an open load turn past 15 minutes of virtual time with no turn end and asserts the prompt is never typed, and rows for a turn start read during the back-off, a busy-session retype, and the run offset and persisted PromptOffset after a hold?"
  - "internal/shuttleengine/doc.go, spec.go, run.go and the loadTurnOpenBound comment: does any text still describe the open-turn case as skipped at an open bound, the timeout as measured from the end of the startup probe, the deadline as moved only by Wait, or the notifier as receiving only a held turn end?"
---

The focus entries name where to look and the question to settle; they do not cap severity.
