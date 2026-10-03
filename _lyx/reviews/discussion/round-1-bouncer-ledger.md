---
round: 1
ledger:
  - key: d7-d9-protected-removal-conflict
    rounds: [1]
    status: resolved
  - key: d3-spawn-import-misses-gitkit-hubforge
    rounds: [1]
    status: resolved
  - key: d3-scan-vs-review-discipline-split
    rounds: [1]
    status: resolved
  - key: d4-websterengine-config-test-tagging
    rounds: [1]
    status: resolved
  - key: scope-items-without-acceptance-criteria
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-bullets-run-together
    rounds: [1]
    status: resolved
  - key: d13-kept-unchanged-belongs-in-support-log
    rounds: [1]
    status: resolved
  - key: perishable-survey-narrative-and-tallies
    rounds: [1]
    status: resolved
  - key: gitkit-repofixture-non-spawning-export
    rounds: [1]
    status: resolved
---

Round 1 raised nine findings against the decision record, all fixed by the round-1 fixer before judging.
The substantive ones were attack-surface gaps: D7 and #325 §5 removed assertions from tests D9 protects without an exception, D3's spawn-import rule let a kit spawn git through `gitkit` or `hubforge`, and D3 never said which Testkit rules a scan enforces and which rest on review.
The rest were coverage gaps in the acceptance criteria, one wrongly tagged spawn-free test in D4, a run-together bullet, and leanness findings that moved D13's kept-unchanged audit and perishable tallies out of the record.
D13's relocation passed the completeness-before-leanness test: the constraints it protected remain in `## Constraints` and the acceptance criteria.
Nothing is open going into round 2.
