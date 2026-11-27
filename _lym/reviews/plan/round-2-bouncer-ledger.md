---
round: 2
ledger:
  - key: off-mode-record-outside-lock
    rounds: [1]
    status: resolved
  - key: prime-window-start-under-next
    rounds: [1]
    status: resolved
  - key: report-update-builder-missing
    rounds: [1]
    status: resolved
  - key: probes-and-recordings-deferred-to-implementation
    rounds: [1]
    status: resolved
  - key: placement-child-registration-and-slug-readers
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: script-runner-calls-unbounded
    rounds: [1]
    status: resolved
  - key: unknown-worktree-row-misses-branch
    rounds: [1]
    status: resolved
  - key: window-command-skipped-on-child-exits
    rounds: [1]
    status: resolved
  - key: tier-table-extends-and-stale-tier-doc
    rounds: [1]
    status: resolved
  - key: mixed-tier-tests-name-no-file
    rounds: [1]
    status: resolved
  - key: query-within-and-locked-places-read-unstated
    rounds: [1]
    status: resolved
  - key: reedcli-window-file-comment-stale
    rounds: [1]
    status: resolved
  - key: overview-package-tree-misses-kwinplace
    rounds: [1]
    status: resolved
  - key: recordopen-keeps-settledby
    rounds: [1]
    status: resolved
  - key: undecodable-places-heal-race
    rounds: [1]
    status: resolved
  - key: vanished-home-branch-untested
    rounds: [1]
    status: resolved
  - key: hidden-verb-bound-incomplete
    rounds: [1]
    status: resolved
  - key: contract-test-covers-part-of-recordings
    rounds: [1]
    status: resolved
  - key: off-mode-report-update-loses-place
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: contract-test-apply-step-skips-by-design
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 opened every key it holds, with its weight in the child flow and planner cards.
Round 2 reports all of them resolved except the placement child's registration scope.

That one stays open: every non-prime child is registered under its worktree's slug, which goes beyond D10.
The reason given for deferring it, that lym has no signal for "inside a run", is disputed by the review,
which names `pauserecord.RunProcessEnv` as that signal.

The round-1 fixes for the report builder and the contract-test coverage each brought in a new defect, opened under new keys:
the builder loses the place under mode `off`, and the contract test skips its apply step by design.

The deferred KWin probes are settled by the added decision entry for a first probe card.
