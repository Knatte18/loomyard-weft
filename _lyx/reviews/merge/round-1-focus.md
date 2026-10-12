---
round: 1
focus:
  - "Where the task's webster pack code (`internal/websterengine`, `internal/planglyph`, `internal/planindex`) touches packages the parent changed (`internal/seatengine/prompt.go`, `internal/shedrun/paths.go`, `internal/reedengine/strand.go`, `internal/testkit/envkit`): does any shared symbol or contract the task relies on differ after the merge?"
  - "The parent's `pattern/PATTERN-daemon-wakeups.md` change adds the status panes' loop in `internal/statuspane/pane.go` to the unbounded-wait list: does any task code add a long-lived wait or poll that the rule's backoff and event-source requirements now cover?"
  - "Test economy: do the task's new `internal/websterengine` and `internal/planglyph` tests or the parent's new tests (`internal/statuspane`, `internal/reedengine`) cover the same behavior twice after the merge, and does any interaction between the two sides lack a covering test?"
---

The overlap note lists no overlapping files or packages, so the interaction is likely thin.
The first entry checks that claim at the symbol level rather than the file level.
