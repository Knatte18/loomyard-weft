---
round: 1
ledger:
  - key: activity-recover-fail-flap
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: past-end-prompt-offset-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW design findings in `internal/shuttleengine/activity.go`.
F1 maps to `activity-recover-fail-flap`: `Recover` runs per part, so a failing pre-offset part flaps the episode.
F2 maps to `past-end-prompt-offset-untested`: the widened past-end clamp has no test row.
Neither is gating, so the loop converges.
