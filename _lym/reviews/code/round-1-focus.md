---
round: 1
focus:
  - "internal/shedverbs/loopwait.go: does the dead-loop stop detail and halt note carry the exit code or signal and the last lines of loop.stderr_path, as the board entry asks, and are its failure paths (missing log, unreadable stderr, no recorded exit) handled without masking the original cause?"
  - "internal/shedverbs/loop.go: does the panic recovery write an envelope on every path, and does the signal marker written before the loop ends by that signal stay correct when the loop is already exiting or the log write fails?"
  - "Board entry's second ask: can a loop exit between Drift-Await done and the Webster start without an envelope, and does the diff close that path, or only report it better?"
  - "Loop log start and exit markers: are the excerpt boundaries correct when several loops share one loop log, so the excerpt holds only the dead loop's lines?"
  - "Test coverage of new behavior in loop_test.go, loopwait_test.go and loop_unix_integration_test.go: is each added behavior (signal marker, panic recovery, excerpt, stderr tail) covered, and does any new test duplicate an existing one?"
  - "internal/shedverbs/doc.go and the commit message and landing summary: does each claim match the diff?"
---

Seed focus drawn from the board entry in the decision record and the diff stat; the record's other sections are empty, so the entry is the measuring stick.
