---
round: 2
ledger:
  - key: card6-wire-origin-read-mutates
    rounds: [1]
    status: resolved
  - key: card3-new-test-vs-extend-lint-comments-test
    rounds: [1]
    status: resolved
  - key: card10-existing-background-files-untargeted
    rounds: [1]
    status: resolved
  - key: card10-documentation-lifecycle-skipped
    rounds: [1]
    status: resolved
  - key: card1-second-top-level-test-false-premise
    rounds: [1]
    status: resolved
  - key: card6-rungate-missing-resign-arrow
    rounds: [1]
    status: resolved
  - key: card3-readoverview-crlf-vs-lf-blob
    rounds: [1]
    status: resolved
  - key: card6-newverifygate-caller-count
    rounds: [1]
    status: resolved
  - key: card6-old-reader-meaning-stated-in-untargeted-files
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card10-uniform-line-ceiling-squeezes-link-gaining-entries
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: card3-test-rename-unlicensed
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: card2-impactsummary-misstates-readers
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card1-in-memory-row-count-wrong
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's eight findings do not recur in round 2, so all are resolved.
Round 2 raised five new findings.
The only design one is the 200-byte line ceiling on card 10's six link-gaining entries, at LOW.
The MEDIUM is a consistency gap: card 6 leaves the old merge-base-reader meaning standing in the shed recipe spec, the `VerifyMergeBase` field comment and the loom recipe comment.
The rest are an unlicensed test rename on card 3, a misstated ImpactSummary on card 2 and a row miscount on card 1.
