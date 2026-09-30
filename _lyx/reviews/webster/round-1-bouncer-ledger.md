---
round: 1
ledger:
  - key: remove-status-strands-printf-warn
    rounds: [1]
    status: resolved
  - key: remove-status-strands-vacuous-guid-pin
    rounds: [1]
    status: resolved
  - key: diff-comments-semantic-line-breaks
    rounds: [1]
    status: resolved
  - key: early-probe-comment-conjunction
    rounds: [1]
    status: resolved
  - key: start-watchdog-test-header-reed-calls
    rounds: [1]
    status: resolved
  - key: step-smoke-timeout-name-covers-start
    rounds: [1]
    status: resolved
  - key: ensure-status-strand-doc-two-calling-verbs
    rounds: [1]
    status: open
---

Round 1 is the first round; there is no prior ledger.

The round-1 review approved with three MEDIUM findings and three NITs.
F1 was printf verbs passed to a slog-style `logger.Warn`, and F2 was the vacuous `wantInLog: "a"` pin that let F1 through.
F3 was fixed-column wrapping in new and changed comments.
F4 was a dropped conjunction, F5 an inaccurate reed-call header, and F6 a timeout name that covered only `step`.
The fixer fixed each one in its own commit, and all six are resolved at HEAD `c7b8cc23d`.

For F3, the fixer stopped rewrapping where a changed sentence meets an unchanged legacy sentence, which leaves a few mid-phrase line ends on touched lines.
CONSTRAINTS.md's "untouched lines are never rewrapped" rule supports that choice, so it is not carried as a finding.

One new non-blocking NIT is open.
`ensureStatusStrand`'s second doc paragraph still cites "the two calling verbs", but only `start` calls it now.
The verdict is APPROVED.
