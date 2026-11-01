---
round: 1
ledger:
  - key: unparsable-description-aborts-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: since-path-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: root-commit-diff-aborts-report
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: last-zero-counts-everything
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: wrapper-headers-omit-recipes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: zero-line-unpriced-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6, in the order of the keys above).
F1 gates: a change description that fails to parse returns an error from `findLanding` and loses the whole report.
F3 is the same all-or-nothing failure for a root commit.
F4 is a behavior change of `-last 0`, F2 and F6 are test gaps, and F5 is stale wrapper header wording.
Each fix text names the exact change, so all are mechanical.
