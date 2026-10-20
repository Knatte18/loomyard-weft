---
round: 3
ledger:
  - key: publish-skip-seam-evaluation-point
    rounds: [1]
    status: resolved
  - key: tree-scope-config-not-open-map
    rounds: [1]
    status: resolved
  - key: d1-category-premise-false
    rounds: [1]
    status: resolved
  - key: tree-key-normalisation-gaps
    rounds: [1, 2]
    status: resolved
  - key: acceptance-criteria-miss-two-behaviours
    rounds: [1]
    status: resolved
  - key: scope-omits-loom-round-gate-change
    rounds: [1]
    status: resolved
  - key: guards-scan-error-undecided
    rounds: [1, 2]
    status: resolved
  - key: d4-rationale-overstates-once
    rounds: [1]
    status: resolved
  - key: editor-path-skips-darn-loader
    rounds: [2]
    status: resolved
  - key: no-targets-refusal-covers-empty-value
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: remove-last-tree-ordering-unstated
    rounds: [2]
    status: resolved
  - key: editor-refusal-keeps-edit-false
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: editor-way-forward-key-unnamed
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: changedpaths-read-shared-unstated
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: guards-parse-kind-covers-unreadable-file
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 settled the gating config shape, the Publish skip seam, D1's category premise, D4's overstatement, the tree-key normalisation gaps, the `--guards` scan error and the last-tree removal order.
Round 2's editor-save finding is resolved: D1 now requires the editor path to run darn's loader.

Round 3 carries no BLOCKING finding.
Its one design finding is new: the editor refusal's kept-edit promise is false against `editHubWide`, and its re-open way forward discards the edit.
The empty-value gap stays open in a new form: the first refusal's "set" must read non-empty so the shipped template and the off switch load.
Three LOW findings are new: the editor message names no key, D2 leaves the changed-paths read's sharing open, and D5's parse kind also covers an unreadable test file.
