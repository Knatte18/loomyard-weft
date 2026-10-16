---
round: 2
ledger:
  - key: verify-omits-tools-sandbox
    rounds: [1]
    status: resolved
  - key: watchdog-nil-branch-ctx-guard-untested
    rounds: [1]
    status: resolved
  - key: down-hubpath-guard-unreachable
    rounds: [1]
    status: resolved
  - key: truncation-test-should-be-a-row
    rounds: [1]
    status: resolved
  - key: looping-rows-vs-empty-logs-assertion
    rounds: [1]
    status: resolved
  - key: stop-affirmative-row-passes-on-wrong-exit
    rounds: [1]
    status: resolved
  - key: watchdog-down-poll-unannotated
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: discover-signal-comments-stale
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: idle-cycles-comment-names-impossible-case
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none BLOCKING; round 2's review raises none of those keys again, so all six are resolved.
Round 2 raised three new findings, none BLOCKING and none `design`.
F1 (`watchdog-down-poll-unannotated`) is a MEDIUM `scope` finding: card 5's 100ms poll constant needs the `//lyx:one-shot` annotation or a file the daemonwakeups scan does not list.
F2 (`discover-signal-comments-stale`) is a LOW `scope` finding about two doc comments card 3 leaves incomplete.
F3 (`idle-cycles-comment-names-impossible-case`) is a LOW `consistency` finding about a comment that names a case that cannot occur.
