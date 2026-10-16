---
round: 1
focus:
  - "internal/orchcli/start.go: does `--continue` resume the recorded session id only through the existing adopt branch, and does every fresh launch record its id and name a replaced id in the hint, as card 3 specifies?"
  - "internal/orchengine/watcher.go and state.go: does the watcher write `session_id` only when the id it reads changes, keep it across `/clear`, and leave a read failure from clobbering a recorded id?"
  - "internal/shuttleengine/sessionread.go and run.go: do the reading and a started run expose the provider session id as card 1 specifies, and are the changed signatures' callers all updated?"
  - "internal/orchcli/start_test.go (large diff) and watcher_test.go: test economy, i.e. new tests that duplicate an existing test's coverage, and added behavior with no coverage."
  - "internal/orchcli/status.go and orchengine/doc.go: does `status` report the recorded id, does `stop` keep it, and do the doc.go changes match the built behavior?"
---

Seed focus from the diff stat against the plan's four cards.
