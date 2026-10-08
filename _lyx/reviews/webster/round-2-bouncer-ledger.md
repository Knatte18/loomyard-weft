---
round: 2
ledger:
  - key: shadow-refresh-misses-late-markers
    rounds: [1]
    status: resolved
  - key: comments-cite-plan-artifacts
    rounds: [1]
    status: resolved
  - key: session-state-test-not-parallel
    rounds: [1]
    status: resolved
  - key: turn-end-api-error-marker-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: shadow-rereads-whole-events-file
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: package-doc-names-claude-hook-event
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings: late markers missed by the shadow refresh, comments citing plan artifacts, and a serial `TestRunner_SessionState`.
The round-2 review raised none of them again, so all three are resolved.
Round 2 raised three new findings.
The turn-end half of the marker re-read has no test row (F1, scope, MEDIUM).
The shadow reads the whole events file each tick instead of from its cursor (F2, design, LOW).
The neutral package doc names Claude's `Notification` hook event (F3, consistency, LOW).
