---
round: 2
ledger:
  - key: card1-misses-run-resume-test-startedfresh
    rounds: [1]
    status: resolved
  - key: card1-tick-table-on-old-rule
    rounds: [1]
    status: resolved
  - key: card1-hold-test-duplicates-existing-coverage
    rounds: [1]
    status: resolved
  - key: card4-newverifygate-error-path-undecided
    rounds: [1]
    status: resolved
  - key: cards10-12-guard-bound-unstated
    rounds: [1]
    status: resolved
  - key: card2-warning-drops-shell-label-and-webstercli-test
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-recover-batch-prefix-still-says-fork
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: answersymbols-classifier-rerun-per-call
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-run-reprompt-wiring-unpinned
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: card9-no-unparseable-constraint-row
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: card6-archiverunafterreset-doc-stale
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1's five keys are resolved: round 2's review found cards 1, 4, 10, 11 and 12 sound on those points.
Round 2 raised six new keys.
Card 2 drops the shell's label, signal and time outstanding from its warning, which breaks an unlisted webstercli test (F1).
Card 6 leaves the fork-only wording in the recover-batch refusal prefix (F2) and in the `ArchiveRunAfterReset` doc (F6).
Card 12's per-call classifier run breaks D6's one-read bound (F3, `design`).
Card 4 does not pin the `Run` wiring of the re-prompt (F4).
Card 9 lacks an unparseable-constraint row (F5).
