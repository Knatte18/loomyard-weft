---
round: 1
focus:
  - "internal/gitexec/gitexec.go stall check: does it fire exactly once at the stall bound, only when the trace file is empty, and stay unarmed when the deadline is at or under the stall bound?"
  - "internal/gitexec/gitexec.go retry path: is a retry made only after a stall kill, never after a deadline kill, git exit or exec failure, with a fresh deadline and stall bound, one Warn, and at most one retry?"
  - "Second stall: does the returned *GitError carry Stalled and Timeout set to the stall bound, so IsTransportFailure in transport.go reports it?"
  - "Trace file handling: created under lyxtemp's run directory, removed on every exit path including kill and retry, and GIT_TRACE_PACKET set only for remote subcommands, with local subcommands getting no trace file and no stall bound?"
  - "Process-group kill: is the stall kill the same path as the deadline kill, with no goroutine or timer leak and no race between the stall check, the deadline and normal completion?"
  - "Tests in gitexec_test.go and export_test.go: does each acceptance criterion have coverage (never-answering remote stalls twice, answering-then-hanging remote hits the deadline unretried, deadline at or under the stall bound, local subcommands), is the stall bound shortened rather than waited out, and does any test duplicate another?"
  - "doc.go: does it describe the stall bound, the retry and the GIT_TRACE_PACKET override, and does the diff stay within the decision record's scope with no call-site changes?"
---

Seed focus from the board entry, the decision record's acceptance criteria and the diff stat.
