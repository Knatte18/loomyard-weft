---
round: 2
ledger:
  - key: landing-run-mapping-conditional-checkpoint
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: tolerant-readers-loosen-live-gates
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: existing-readers-vs-git-object-reads
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: batten-record-bound-cleanup-commit
    rounds: [1]
    status: resolved
  - key: land-ready-notice-left-open
    rounds: [1]
    status: resolved
  - key: halt-note-source-and-unmatched
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: absent-vs-zero-composite-figures
    rounds: [1]
    status: resolved
  - key: missing-acceptance-criteria
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: direct-pushes-outside-fix-share
    rounds: [1]
    status: resolved
  - key: escalation-circling-precedence
    rounds: [1]
    status: resolved
  - key: flag-bounds-default-range-only
    rounds: [1]
    status: resolved
  - key: support-log-q7-stale
    rounds: [1]
    status: resolved
  - key: rationale-argues-rejected-alternatives
    rounds: [1]
    status: resolved
  - key: hand-landing-inferred-from-land-row
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: batten-span-own-fresh-seed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 2 raised seven findings, F1 to F7.
Eight round-1 keys did not recur and are marked resolved.
Five round-1 keys recur: F1 on the landing join, F3 on the tolerant parse-from-bytes rule, F4 on the verdict reader, F6 on the halt-note source, F7 on acceptance criteria.
F2 (`hand` inferred from batten's Land row) and F5 (which fresh seed is a tag's own) are new.
Both BLOCKING findings are F1 and F3, in the same two Decisions, D10 and D5/D9, as in round 1.
