---
round: 1
ledger:
  - key: gitkit-gitrepo-untagged-testmain
    rounds: [1]
    status: resolved
  - key: per-cli-copies-missed-tests
    rounds: [1]
    status: resolved
  - key: stencil-seeders-outside-cards-21-22
    rounds: [1]
    status: resolved
  - key: batten-commit-prefix-unpinned
    rounds: [1]
    status: resolved
  - key: interruptpolicy-meta-reads-loomrowengines
    rounds: [1]
    status: resolved
  - key: claims-table-section-scope
    rounds: [1]
    status: resolved
  - key: card25-targets-without-stencil-wording
    rounds: [1]
    status: resolved
  - key: card18-placeholder-writers
    rounds: [1]
    status: resolved
  - key: tmux-open-ended-fixes-untargeted
    rounds: [1]
    status: resolved
  - key: removal-let-through-unstated-24-25-27
    rounds: [1]
    status: resolved
  - key: d23-kit-package-docs
    rounds: [1]
    status: resolved
  - key: tmuxkit-61-byte-bound-unguarded
    rounds: [1]
    status: resolved
  - key: statuscommit-marker-hook-unnamed
    rounds: [1]
    status: resolved
  - key: noforceadd-scan-unconverted
    rounds: [1]
    status: open
  - key: removed-names-referenced-outside-targets
    rounds: [1]
    status: open
---

Round 1's report raised thirteen findings, F1–F13.
Each was legitimate, and the fixer resolved each one in the plan; I checked every fix against the tree.
That includes a full listing of tagged test packages against cards 30–32's `TestMain` coverage.

Two new findings stay open.
First, `internal/gitrepo/noforceadd_test.go` scans source for the Never Force-Add Invariant, so D2 binds it, yet no card converts it.
Card 10 is the natural home.
Second, card 15 removes `loomRowEngines`, which production `internal/loomrecipe/names.go:18` still names in a doc comment.
Card 11 deletes `drift_test.go` and `longlist_test.go`, which comments in `cmd/lyx/seamsignature_test.go:7` and `cmd/lyx/sandbox_coverage_test.go:39` still name.
None of those three files is a target of its card.
