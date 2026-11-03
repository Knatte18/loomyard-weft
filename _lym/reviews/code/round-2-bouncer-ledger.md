---
round: 2
ledger:
  - key: baseline-prompt-names-old-dir-base
    rounds: [1]
    status: resolved
  - key: pairdir-test-asserts-old-seam
    rounds: [1]
    status: resolved
  - key: hubwide-editor-save-missing-dir-untested
    rounds: [1]
    status: resolved
  - key: worktreedir-wiring-unpinned
    rounds: [1]
    status: resolved
  - key: checkdirs-compare-entry-unpinned
    rounds: [1]
    status: resolved
  - key: side-capture-refused-with-baseline-way-forward
    rounds: [1]
    status: resolved
  - key: driftcli-doc-keeps-old-base
    rounds: [1]
    status: resolved
  - key: statcommanddir-file-not-dir-untested
    rounds: [1]
    status: resolved
  - key: resolvecommanddir-one-rule-claim
    rounds: [1]
    status: resolved
  - key: loader-refusal-reading-unrecorded
    rounds: [1]
    status: resolved
  - key: per-helper-request-test-duplicates-integration
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised ten findings, one gating.
Round 2's review cleared each of them in its verdicts on the focus items, so all ten are resolved.
Round 2 raised one new finding, F1, a LOW `consistency` per-helper request test in `internal/loomcli/driftwiring_test.go` that the integration test already covers; its fix is a deletion the finding text fully specifies.
