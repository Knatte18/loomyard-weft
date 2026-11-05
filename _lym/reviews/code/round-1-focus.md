---
round: 1
focus:
  - "internal/shuttleengine/session.go `Runner.SessionIdle`: does the recordless-strand engine pick match `Runner.Send`'s provider-tag route in every case, including a missing or unknown tag, and is a failed engine lookup handled?"
  - "internal/shuttleengine/codexengine/events.go, rollout.go and codexengine.go: does a Codex exit now reach `lym shuttle state` as ended, and does a loop that ended while Codex is still live, or a run that exits cleanly, read correctly?"
  - "internal/shuttleengine/codexengine/command.go and internal/shell/*: how is the sandbox opened for the creel socket? Is it the narrowest opening that works, is it quoted correctly for both posix and pwsh, and does the `shell` package change keep the other callers' behavior?"
  - "internal/shuttleengine/codexengine/doc.go: does it say what is opened and why, as the Goal's third fault requires, and does the diff leave the creel-read and post-reply claim true?"
  - "Tests in session_test.go, shell_test.go and the codexengine tests: does each of the three faults have coverage of its behavior, and does any test duplicate an existing one or assert static prose?"
---

The Goal's three faults each map to one area above.
The record's other sections are empty, so the Goal is the only measuring stick.
