---
round: 3
ledger:
  - key: landing-run-mapping-conditional-checkpoint
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: tolerant-readers-loosen-live-gates
    rounds: [1, 2]
    status: resolved
  - key: existing-readers-vs-git-object-reads
    rounds: [1, 2]
    status: resolved
  - key: batten-record-bound-cleanup-commit
    rounds: [1]
    status: resolved
  - key: land-ready-notice-left-open
    rounds: [1]
    status: resolved
  - key: halt-note-source-and-unmatched
    rounds: [1, 2]
    status: resolved
  - key: absent-vs-zero-composite-figures
    rounds: [1]
    status: resolved
  - key: missing-acceptance-criteria
    rounds: [1, 2, 3]
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
    status: resolved
  - key: batten-span-own-fresh-seed
    rounds: [2]
    status: resolved
  - key: time-join-reused-slug-window
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: fast-forward-landings-flood-unmapped
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: window-exclusion-criterion-overclaims
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 3 raised five findings, F1 to F5.
F1 stays on the D10 landing join, now at its untagged time-join branch, and is still BLOCKING after three rounds.
F5 is the third round of the acceptance-criteria gap.
F2 (reused-slug window bounds), F3 (fast-forward landings) and F4 (window-exclusion criterion) are new keys.
The tolerant-reader, verdict-reader, hand-test-from-Land-row and fresh-seed keys did not recur; the review's focus verdicts record them as settled, and they are marked resolved.
The halt-note key was not raised and is marked resolved.
