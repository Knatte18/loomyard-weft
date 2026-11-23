---
round: 1
ledger:
  - key: executable-blobs-skipped-in-base-checkout
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: hard-coded-dotnet-scoper-branches
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: no-test-run-reads-as-green
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: restore-or-outside-build-failure-exits-terminally
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: per-project-grouping-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: checkout-root-swap-only-in-skipping-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: mutation-survivors-method-end-and-filter-terms
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-branches-without-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tier-failure-names-non-tier-rule-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: judge-test-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: nearest-project-drops-readdir-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: exit-contract-names-only-go-label-form
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-unjudged-list-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised thirteen findings, F1 to F13, in the order above.
F1 and F2 are BLOCKING design findings: executable files missing from the base checkout, and new `"dotnet"` literal branches in `internal/gatecli/redfirst.go`.
F3 and F5 are gating MEDIUM design findings.
F4 is a MEDIUM scope finding on the restore and build-failure exit.
The rest are LOW test-coverage, doc and consistency items.
F2, F3 and F4 are marked `design-wrong` because their fixes change an interface or a contract; the others are local corrections the finding text already fixes.
