---
round: 3
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
    status: resolved
  - key: d7-false-flag-bound-wrong
    rounds: [1, 2]
    status: resolved
  - key: d7-failure-paths-incomplete
    rounds: [1]
    status: resolved
  - key: d7-repeat-run-and-budget-understated
    rounds: [1]
    status: resolved
  - key: d6-refresh-child-no-owner
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: report-recorded-set-vs-d8-match-set
    rounds: [2]
    status: resolved
  - key: d7-step1-renamed-moved-deleted-test-files
    rounds: [2]
    status: resolved
  - key: d3-ordinal-bound-understated
    rounds: [2]
    status: resolved
  - key: d5-bound-misses-non-go-files
    rounds: [2]
    status: resolved
  - key: spawn-scan-home-unstated-after-d1-move
    rounds: [2]
    status: resolved
  - key: d7-criterion-contradicts-step5
    rounds: [2]
    status: resolved
  - key: d7-step4-build-and-slot-wait-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-followup-report-channel-unnamed
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: keep-marker-scan-not-importable
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-d5-go-file-sets-differ
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-git-reads-no-fabric-home
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised 17 and 10 findings.
Round 3 raised five, none BLOCKING, and none on a key from the earlier rounds.
The recurring keys (`d6-refresh-child-no-owner`, `d7-false-flag-bound-wrong`, `testmap-no-per-test-timeout`) and every other earlier key are resolved: the review re-read the round-2 fixes and found no regression.

Open at the close of round 3:

- F1 (MEDIUM, design): D7 step 4 states no bound on coverage builds or its gate-slot wait, and the rationale's cost line understates the worst case.
- F2 (LOW, design): D6's follow-up refresh rests on a report channel the record does not name.
- F3 (LOW, design): the keep-marker scan stays in the unimportable `cmd/testtiming` package.
- F4 (LOW, consistency): D4 hashes every Go file in the directory while D5 compares only the tracked set.
- F5 (LOW, design): D5's git reads at HEAD have no home under the fabric boundary.
