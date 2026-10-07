---
round: 5
ledger:
  - key: receiving-set-misses-webster-and-burler-runner-sites
    rounds: [4, 5]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-names-batten-site-with-no-runner
    rounds: [4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: scope-says-once-vs-once-per-held-turn-end
    rounds: [4, 5]
    status: open
    class: consistency
    severity: LOW
  - key: notifier-receiving-set-per-spawner-not-runner
    rounds: [4]
    status: resolved
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

Findings fell from ten in round 1 to two in round 4, then three in round 5 with one MEDIUM; the highest severity has been MEDIUM since round 2.

Round 4's `notifier-receiving-set-per-spawner-not-runner` is resolved: D5 now speaks per runner.
Round 5 finds the runner basis incomplete, so it opens `receiving-set-misses-webster-and-burler-runner-sites` (MEDIUM design, F1): `webstercli` and `burlercli` build their own runners, and a `recover-batch` recovery strand sits in neither the notifying set nor the wire-none list.
The fix is to state the set once by constructing package in D5, refer to it from the Auto-mode assumption, and decide the two sites.

Open and non-gating:
- `d5-names-batten-site-with-no-runner` (LOW consistency, F2): `battencli` and `battenshed` build no runner, so the clause naming batten-driven runs goes.
- `scope-says-once-vs-once-per-held-turn-end` (LOW consistency, F3): Scope and the D5 title omit the Attach exception, and no acceptance criterion pins an Attach over several held turn ends notifying one.
