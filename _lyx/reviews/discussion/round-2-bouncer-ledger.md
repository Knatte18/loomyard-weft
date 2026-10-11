---
round: 2
ledger:
  - key: foreign-box-not-nothing-typed
    rounds: [1]
    status: resolved
  - key: start-delivery-not-retried
    rounds: [1]
    status: resolved
  - key: placeholder-acceptance-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: idle-notice-retry-unseen
    rounds: [1]
    status: resolved
  - key: unreadable-box-after-paste-submit
    rounds: [1]
    status: resolved
  - key: foreign-box-error-identity
    rounds: [1]
    status: resolved
  - key: bracketed-paste-assumed
    rounds: [1]
    status: resolved
  - key: unknown-readiness-box-bound
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: tmux-invocation-undercount
    rounds: [1]
    status: resolved
  - key: draft-text-in-orch-notice
    rounds: [1]
    status: resolved
  - key: d8-dedupe-resume-line-unstable
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: not-landed-may-have-landed-no-record
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: foreign-box-start-dies-at-once
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: send-hold-stuck-reason-flaps
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eleven findings, led by the BLOCKING foreign-box delivery semantics.
Round 2 no longer raises ten of them, so they are resolved; only the placeholder acceptance remains, now LOW.
Round 2 adds four new keys: the unstable resume line under D8's dedupe (F1), the not-landed outcome with no pending-send record (F2), the foreign-box start dying at once rather than after the hold (F4), and the held-notice reason clearing on the watcher's probe (F5).
F1 and F2 are MEDIUM `design` and first raised in round 2, so Go carries them into `## Open risks`.
