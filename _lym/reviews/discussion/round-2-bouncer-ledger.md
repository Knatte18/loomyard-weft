---
round: 2
ledger:
  - key: reports-step-unreachable-on-tuned-hub
    rounds: [1]
    status: resolved
  - key: d7-base-text-source-and-no-parent
    rounds: [1]
    status: resolved
  - key: d5-skips-retrigger-refresh-every-tick
    rounds: [1]
    status: resolved
  - key: d5-untracked-file-stale-and-skipped
    rounds: [1]
    status: resolved
  - key: d5-landing-under-running-refresh
    rounds: [1]
    status: resolved
  - key: testmap-no-per-test-timeout
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-false-flag-bound-wrong
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-failure-paths-incomplete
    rounds: [1]
    status: resolved
  - key: d7-repeat-run-and-budget-understated
    rounds: [1]
    status: resolved
  - key: d6-refresh-child-no-owner
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-keep-escape-unbounded
    rounds: [1]
    status: resolved
  - key: d7-step2-drops-by-unknown-results
    rounds: [1]
    status: resolved
  - key: d8-red-report-match-undefined
    rounds: [1]
    status: resolved
  - key: d7-changed-files-line-mixes-edits-and-drift
    rounds: [1]
    status: resolved
  - key: prime-inplace-update-fact-under-assumptions
    rounds: [1]
    status: resolved
  - key: fewest-covering-tests-claim
    rounds: [1]
    status: resolved
  - key: d8-map-rebuild-bound-missing
    rounds: [1]
    status: resolved
  - key: d5-no-binary-package-no-map-loop
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: report-recorded-set-vs-d8-match-set
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-step1-renamed-moved-deleted-test-files
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d3-ordinal-bound-understated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-bound-misses-non-go-files
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: spawn-scan-home-unstated-after-d1-move
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-criterion-contradicts-step5
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 17 findings. Round 2 re-raised three of those keys and raised seven new ones; the other round-1 keys no longer appear and are resolved.

Open at the close of round 2:

- F1 (BLOCKING, design): a package with no test binary or a failed coverage build gets no map, so it stays stale and D6 restarts a refresh on every tick.
- F2 (MEDIUM, design, `d6-refresh-child-no-owner`): D6's dobby-stop claim contradicts `runproc.EndRole`'s contract.
- F3 (MEDIUM, design): the report records function hashes only for judged tests, while D8's match compares step 1's whole set.
- F4 (MEDIUM, design, `testmap-no-per-test-timeout`): D5's time bound omits the slot wait and any build timeout, and a timed-out test's descendant can hold the output pipe.
- F5 to F10 (LOW): renamed or deleted test files in step 1, the D3 ordinal bound, non-Go files in the D5 bound, the D7 false-cover bound (`d7-false-flag-bound-wrong`), the spawn scan's home after D1's move, and a D7 criterion that contradicts step 5.
