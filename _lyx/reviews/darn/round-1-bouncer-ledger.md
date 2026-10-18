---
round: 1
ledger:
  - key: renewed-hold-day-rollover-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: hold-after-expiry-event-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW coverage findings in `internal/creelengine` tests.
F1 maps to `renewed-hold-day-rollover-untested`: no test carries a renewed hold across a day rollover.
F2 maps to `hold-after-expiry-event-untested`: no test pins that a hold taken after expiry journals `held`.
Neither is gating.
