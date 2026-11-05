---
round: 3
ledger:
  - key: judge-snapshot-counts-loom-status-commits
    rounds: [1, 2]
    status: resolved
  - key: duplicate-delete-verifyfixture-helpers
    rounds: [1]
    status: resolved
  - key: cards-4-16-26-miss-test-members
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: drift-expect-row-name-repeated-literal
    rounds: [2]
    status: resolved
  - key: judgement-yaml-never-committed-or-removed
    rounds: [2]
    status: resolved
  - key: overview-darn-sentence-lacks-drift-expect
    rounds: [2]
    status: resolved
  - key: drift-expect-never-counted-stuck-untested
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: drift-verify-interim-command-stuck-untested
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card-16-needs-row-constants-created-by-card-19
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: judge-snapshot-exempt-set-unbounded
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parser-refusals-without-test-rows
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings and round 2 four more; round 3's review clears every key open after round 2.
Its Rejected section records checks of the test members on the removal cards, the capture lock-wait sleep, the `Drift-Expect` row constant, the snapshot exempt set against loom's status commits, `judgement.yaml` removal and the overview and docs sentences.

Round 3 raises five new keys.
Two are BLOCKING scope findings, both prescribed refusals that name `lym drift defer` with no test row reaching them: the `Drift-Expect` record with no `awaited_at`, and card 16's interim `command` diff mode.
One is a MEDIUM design finding: card 16 compares `Deps.Rows` against `darnWriteRow` and `websterRow`, which card 19 creates three cards later.
The last two are LOW: the judge snapshot's exempt paths state no bound, and the parser refusals of cards 5 and 6 have no rows in their tests.
The review text fixes each result, so all five are mechanical.
