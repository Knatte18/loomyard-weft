---
round: 2
focus:
  - "In the `Long` help of `statusCmd` in `internal/orchcli/status.go`: is every `strand_down` value rendered from its named constant in `internal/orchroute/live.go` or `internal/reedengine/apply.go`, and does the list match the values the command can report?"
  - "In `downCause` (`internal/reedengine/apply.go`), `strandsElsewhereLocked` (`internal/reedengine/ownwindow.go`) and `Status` (`internal/reedengine/lifecycle.go`): which cause does each own-window strand read when the server-wide pane lookup fails, does every consumer of the cause handle each value, and do the cause's doc comment, the help and the change description agree with the code?"
  - "In `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` (`internal/reedengine/lifecycle_test.go`): are the down causes pinned for the failed-lookup case and for a strand bound to no pane, with expected values stated independently of the code under test?"
  - "In `Live` (`internal/orchroute/live.go`): do both log branches emit the same message, and does that message match what `_lym/landing/summary.md` tells a reader to search for?"
---
