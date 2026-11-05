---
round: 2
ledger:
  - key: judge-snapshot-counts-loom-status-commits
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: duplicate-delete-verifyfixture-helpers
    rounds: [1]
    status: resolved
  - key: cards-4-16-26-miss-test-members
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: drift-amend-refusal-no-way-forward
    rounds: [1]
    status: resolved
  - key: check-before-expect-names-defer
    rounds: [1]
    status: resolved
  - key: checkbaseline-unguarded-hides-missing-dir
    rounds: [1]
    status: resolved
  - key: card-18-names-removed-verbs-in-headers
    rounds: [1]
    status: resolved
  - key: capture-lock-wait-no-cadence-no-pattern-listing
    rounds: [1]
    status: resolved
  - key: stale-docs-drift-rows-and-verbs
    rounds: [1]
    status: resolved
  - key: defer-held-lock-wrong-way-forward
    rounds: [1]
    status: resolved
  - key: countedcapture-left-without-caller
    rounds: [1]
    status: resolved
  - key: capture-current-lock-wait-no-injected-sleep
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: drift-expect-row-name-repeated-literal
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: judgement-yaml-never-committed-or-removed
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: overview-darn-sentence-lacks-drift-expect
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings; round 2 clears nine of them.
Two persist: the test members missing from the Edit lists of the removal cards, now on cards 24, 25 and 26, and the judge snapshot, whose exempt weft set is still narrower than what loom's commits stage.
The fixers' round 1 changes introduced no recurrence of the other nine keys.

Round 2 adds four keys: the current capture's lock wait lacking an injected sleep (BLOCKING design), the `Drift-Expect` row name repeated as a literal (BLOCKING consistency), `judgement.yaml` left untracked after the judge returns (LOW decision), and the overview's darn recipe sentence omitting `Drift-Expect` (LOW consistency).
Only the judge snapshot fix changes an approach, by judging weft paths by content, so it stays `design-wrong`; the rest are local corrections the finding text fixes.
