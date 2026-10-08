---
round: 3
ledger:
  - key: d1-earlier-round-source-unspecified
    rounds: [1]
    status: resolved
  - key: d2-entry-identity-across-generations
    rounds: [1]
    status: resolved
  - key: d2-commit-route-plan-review
    rounds: [1]
    status: resolved
  - key: d2-write-edge-unbounded
    rounds: [1]
    status: resolved
  - key: goal-omits-round-floor
    rounds: [1]
    status: resolved
  - key: carry-over-meaning-only-in-support-log
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
  - key: carry-over-catch-not-wired-to-consumers
    rounds: [2]
    status: resolved
  - key: d1-degraded-ledger-read-weakens-guard
    rounds: [2]
    status: resolved
  - key: d2-entry-format-and-extent-unfixed
    rounds: [2]
    status: resolved
  - key: d2-bound-leans-on-validate
    rounds: [2]
    status: resolved
  - key: docs-constraint-omits-discussionparser
    rounds: [2]
    status: resolved
  - key: d2-path-form-and-rounds-source-open
    rounds: [2]
    status: resolved
  - key: d2-stale-entry-lifetime-unstated
    rounds: [2]
    status: resolved
  - key: d2-marker-recognition-unspecified
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-bound-byte-check-tautological
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: goal-claims-consumers-that-do-not-read-record
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
---

Round 1's seven findings and round 2's seven findings no longer appear in round 3's review, so all fourteen keys are resolved.
Round 3 raises three new `design` findings at MEDIUM, mapped to the keys above: marker recognition (F1), the tautological byte-comparison bound (F2), and the Goal's claim about consumers (F3).
The finding count fell from seven to three, and no key recurs.
