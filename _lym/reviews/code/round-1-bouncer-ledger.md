---
round: 1
ledger:
  - key: baseline-prompt-names-old-dir-base
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pairdir-test-asserts-old-seam
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: hubwide-editor-save-missing-dir-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: worktreedir-wiring-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: checkdirs-compare-entry-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: side-capture-refused-with-baseline-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: driftcli-doc-keeps-old-base
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: statcommanddir-file-not-dir-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: resolvecommanddir-one-rule-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loader-refusal-reading-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 raised ten findings; all are open, none recur from an earlier round.
The one gating finding is the Drift-Baseline prompt text that still describes the old directory base.
The remaining MEDIUM findings are test-pinning gaps (PairDir seam, hub-wide editor save, `WorktreeDir` wiring, the `compare` entry of `CheckDirs`).
The LOW findings are a side-capture refusal wording, two doc claims, a missing file-not-directory row, and an unrecorded reading of "loader" that goes to the parent.
