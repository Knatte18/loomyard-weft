---
round: 1
ledger:
  - key: blamed-producer-can-go-blank
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: classifyhalt-narrower-than-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: backoff-schedule-and-remote-path-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: spawn-refusal-window-expiry-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: interrupted-stop-empty-unreadable-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loopexitstop-pause-reason-value-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loomafterinterrupt-failed-fallback-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: paused-guard-on-pause-reason-untested
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised eight design findings, all LOW or NIT: two correctness gaps against the decision record (F1 blank blamed producer, F2 narrow `ClassifyHalt`) and six coverage gaps (F3 to F8).
No finding is gating, so the loop converges.
