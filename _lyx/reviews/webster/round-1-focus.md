---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/background.go: does `outstandingBackgroundTasks` read the transcript only when `background_tasks` is absent or not a list, and does `transcriptHasUnmatchedLaunch` treat `queue-operation` and `queued_command` lines as completions under the same after-the-tool-result ordering rule?"
  - "internal/shuttleengine/wait.go and run.go: is every turn end without all output files held, on every path (Stop, live ask, expired shell), with the events-file offset computed once in `pollEventsTick` and reused by `expiredTurnEnd`; does a hold never extend `run.deadline`; and does `finalize` no longer write `AskingOffset`?"
  - "internal/shuttleengine/hold.go and the notifier call in Wait: does the notice follow the plan's shape (prefix, strand name, tail quoted, at most five tasks, message start last), with every agent-written part inside `«»`, stripped of control characters and delimiters, and cut to 200 runes; is `NotifiedOffset` persisted before the notifier call; and is the notice once per held turn end, including after an Attach replay?"
  - "Removal order across cards 5, 9, 10, 11 and 12: is every `OutcomeAsking` and `AwaitOperator` use gone in the end state (shed, websterengine, treadle, CLI help, comments, docs), with the string `asking` surviving only as the unexported attach-side constant, and does each commit build and pass tests on its own?"
  - "Attach on legacy `asking` records (card 7) and the notifier wiring in loomcli, webstercli and burlercli (card 8): does Attach wait on a live strand instead of respawning beside it, and does each runner notify the prime's orch through the queue?"
  - "Per-card mechanical checks and test economy: for each card with a Rename, Delete or similar group, did the type-specific check run against every target of that group; are the rewritten tests the ones the plan names, rather than new tests added beside them; and do the replay corpus and live scenarios (cards 14 to 16) cover behavior the earlier cards' tests do not already cover?"
  - "Doc comments and `doc.go` changes in every touched package: do they follow the repo's CLAUDE.md and PATTERN.md conventions (semantic line breaks, godoc rules, the doc landing in the same commit as the behavior)?"
---

Review range: no rework round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main`.
The entries name where to look and which question to settle; none pre-states a verdict.
