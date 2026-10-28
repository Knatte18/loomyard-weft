---
round: 1
focus:
  - "internal/shuttleengine/activity.go: D3 says a running run with a dead waiter yields a reading with LastActivity only and zero API-error and usage-limit parts; check every path through ReadAgentActivity and readRunActivity zeroes those parts for a waiterless run, and that the waiterless episode is still logged once."
  - "internal/shuttleengine/carryover.go: D5 and D6 select the predecessor run (same role key; stopped, died or timeout, or running with waiter gone) and compute operator instructions (turn starts past the prompt offset, minus the first, minus sent texts); check the selection order, the empty role key and done-predecessor bounds, dedup without repeats, and failure paths for a missing or unreadable run directory or record."
  - "internal/shuttleengine/run.go and sessionsignal.go: D4 appends each verified send's text to the sent-texts file before typing, and D6 appends the carried block to the new prompt before the orphan sweep and records it in the new run's record; check ordering, that a failed append does not block the send, and that the notifier line is sent once and also logged at Warn."
  - "internal/shuttleengine/claudeengine/settings.go, signals.go and the settings golden: D1 stamp command on PostToolUse, PreCompact and PostCompact, in a hub and outside one, with the creel mail hook still riding PostToolUse in a hub; D5 empty instruction for a <task-notification> and a slash-command prompt; D7 and D8 no subagent hook and no auto-continue key. Check the golden diff matches only the intended additions."
  - "PATTERN invariants named in the decision record (provider seam, tolerant records, test speed, daemon wakeups): check hook names and the task-notification marker stay inside claudeengine, new run-record fields are optional so an older record still reads, and no new wait or poll is added."
  - "Docs and change description: the doc.go changes cover the new module behavior, and _lym/landing/summary.md, when present, names SubagentStart, SubagentStop and the auto-continue setting as left out with their reasons, and claims nothing the diff does not bear out."
  - "Test economy: the new activity, carryover, settings and signals tests each cover a decision's acceptance criterion; flag duplicates of an existing test's coverage or a behavior the diff adds with no test."
---

Focus covers each decision D1 to D8 against its acceptance criterion, with extra weight on the carry-over selection and the waiterless reading, where the failure paths are most numerous.
