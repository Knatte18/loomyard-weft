---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
  - key: d1-degraded-ledger-read-weakens-guard
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-entry-format-and-extent-unfixed
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d2-bound-leans-on-validate
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: docs-constraint-omits-discussionparser
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d2-path-form-and-rounds-source-open
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: d2-stale-entry-lifetime-unstated
    rounds: [2]
    status: open
    class: decision
    severity: LOW
---

Round 1's seven findings no longer appear in round 2's review, and its focus checks confirm D1's source, the generation and ordering of D2's write, the commit route and the Goal's rule as settled, so those keys are resolved.
Round 2 raises seven new findings, none recurring.
Three are gating `design` findings at MEDIUM: the carry-over catch that no consumer is told to read (F1), D1's guard weakening on an unreadable earlier ledger (F2), and D2's bound resting on `Validate` (F4).
The rest are D2's entry format (F3, MEDIUM `decision`), the docs list (F5), the path form and rounds source (F6) and the stale-entry lifetime (F7), all non-gating.
