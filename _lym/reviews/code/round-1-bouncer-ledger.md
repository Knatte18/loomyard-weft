---
round: 1
ledger:
    - key: strand-down-help-literal-causes
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: failed-pane-lookup-misreports-down-cause
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: unbound-strand-down-cause-unpinned
      rounds:
        - 1
      status: open
      class: scope
      severity: LOW
      fix: mechanical
    - key: down-reading-log-message-duplicated
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 is the first round.
The one gating finding is the `strand_down` help in `internal/orchcli/status.go` repeating cause values that are named constants.
The other findings are about the cause reported after a failed server-wide pane lookup,
a missing test row for a strand bound to no pane,
and the down-reading log message spelled twice in `internal/orchroute/live.go`.
The failed-lookup fix adds a reported cause value, so its new text needs a fresh review.
