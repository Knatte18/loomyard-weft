---
round: 1
ledger:
  - key: reports-step-unreachable-on-tuned-hub
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-base-text-source-and-no-parent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-skips-retrigger-refresh-every-tick
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-untracked-file-stale-and-skipped
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d5-landing-under-running-refresh
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: testmap-no-per-test-timeout
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-false-flag-bound-wrong
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d7-failure-paths-incomplete
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-repeat-run-and-budget-understated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-refresh-child-no-owner
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d7-keep-escape-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-step2-drops-by-unknown-results
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-red-report-match-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-changed-files-line-mixes-edits-and-drift
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prime-inplace-update-fact-under-assumptions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fewest-covering-tests-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: d8-map-rebuild-bound-missing
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised 17 findings, all open.
The gating ones are F1 (BLOCKING: the `reports` step never reaches a hub whose base `gate.yaml` sets the four keys whole), F2 (no named source for function text at the merge base), F3 (skipped packages retrigger the refresh), F5 (a landing under a running refresh), F6 (no per-test timeout in `lym gate testmap`) and F7 (the false-flag bound is wrong).
F4 (untracked files both stale and skipped) is MEDIUM consistency.
The rest are LOW or NIT: failure paths, repeat cost, dobby child ownership, the keep escape, step ordering, the red-report match, the changed-files line, a fact filed as an assumption, a cover-count wording and the map-rebuild bound.
