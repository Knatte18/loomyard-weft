---
round: 1
ledger:
  - key: empty-landing-parent-records-commit-counts-as-changed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: empty-diff-test-lacks-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: restep-after-failed-push-overwrites-empty-flag
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-follow-up-on-empty-landing-line-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: report-dir-accepts-records-tree
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darn-spec-test-pins-stencil-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: report-passage-error-path-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, all open.
The two gating ones are F1 (parent records commit makes `landedDiffEmpty` see a records path, so the flag never fires in a one-repository hub) and F3 (a re-step after a failed parent push recomputes the flag as empty).
F2 is a BLOCKING test-speed finding on the new Finalize test.
F4 to F7 are LOW test-coverage and validation gaps in dobby's landing notice and the darn report passage.
