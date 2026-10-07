---
round: 3
ledger:
  - key: hold-notice-task-label-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: hold-notice-quote-undelimited
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: deadline-bound-false-for-autonomous-callers
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: notified-spawner-set-differs
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
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

Round 1 raised ten findings, round 2 five, round 3 four; the highest severity fell from BLOCKING to MEDIUM and stayed there.

Resolved by round 3:
- The transcript-fallback wording (D2, Open risks, acceptance criteria) now agrees, and the legacy replay backlog is decided as one notice for the last held turn end.
- The incident narrative in D1 to D3 no longer draws a finding.
- The quote's control-character replacement and character cut hold, so the round-2 agent-text key is closed; the remaining gaps are new keys below.

Open in round 3:
- `hold-notice-task-label-unbounded` (MEDIUM design): task labels are raw agent-written shell commands, uncut and possibly multi-line, so `QueueNotice` refuses the line and the one notice is lost; the label is also agent text with no bound.
- `hold-notice-quote-undelimited` (LOW decision): the quote has no fixed delimiter and sits before the way-forward text.
- `deadline-bound-false-for-autonomous-callers` (LOW consistency): D4 says the deadline comes from `*_timeout_min` config, but `--timeout`, a recipe row `timeout_s` and a treadle profile value also set `Spec.Timeout`.
- `notified-spawner-set-differs` (NIT consistency): the assumptions and D5 name different spawner sets.
