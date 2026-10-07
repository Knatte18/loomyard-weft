---
round: 4
ledger:
  - key: notifier-receiving-set-per-spawner-not-runner
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
  - key: scope-says-once-vs-once-per-held-turn-end
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
  - key: hold-notice-task-label-unbounded
    rounds: [3]
    status: resolved
  - key: hold-notice-quote-undelimited
    rounds: [3]
    status: resolved
  - key: deadline-bound-false-for-autonomous-callers
    rounds: [1, 2, 3]
    status: resolved
  - key: notified-spawner-set-differs
    rounds: [3]
    status: resolved
  - key: hold-notice-agent-text-unbounded
    rounds: [1, 2]
    status: resolved
  - key: d2-fallback-contradicts-open-risks
    rounds: [1, 2]
    status: resolved
  - key: legacy-attach-and-notice-persistence-open
    rounds: [1, 2]
    status: resolved
  - key: incident-narrative-in-d1-d3
    rounds: [1, 2]
    status: resolved
  - key: d9-notice-lacks-outstanding-tasks
    rounds: [1]
    status: resolved
  - key: d8-tail-check-whitespace-unspecified
    rounds: [1]
    status: resolved
  - key: scope-omits-in-scope-decisions
    rounds: [1]
    status: resolved
  - key: decisions-without-acceptance-criteria
    rounds: [1]
    status: resolved
  - key: requirements-only-in-support-log
    rounds: [1]
    status: resolved
---

Findings fell from ten in round 1 to five, four and two in round 4; the highest severity has been MEDIUM since round 2.

Resolved in round 4:
- The label channel and the quote delimiter and position are bounded, which closes `hold-notice-task-label-unbounded` and `hold-notice-quote-undelimited`.
- D4's deadline source is true for every caller, which closes `deadline-bound-false-for-autonomous-callers`.
- The two spawner-set wordings now match, which closes `notified-spawner-set-differs`; F1 below replaces it with a deeper problem.

Open in round 4:
- `notifier-receiving-set-per-spawner-not-runner` (MEDIUM design, F1): the notifier is a `Runner` seam, but D5 and the Auto-mode assumptions word the receiving set per spawner.
  The loom wiring's one runner also carries friction reflection and merge-resolve, so they would notify, and "batten-driven" is named as a wiring site though `battencli` builds no runner.
- `scope-says-once-vs-once-per-held-turn-end` (NIT consistency, F2): Scope says the parent is notified once, D5 says once per held turn end.
