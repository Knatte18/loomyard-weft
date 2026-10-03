---
round: 2
ledger:
  - key: pointer-launch-breaks-driverbootstrap-stub
    rounds: [1]
    status: resolved
  - key: pointer-launch-removed-guard-unbounded
    rounds: [1]
    status: resolved
  - key: verify-parity-is-source-scan-not-verdict
    rounds: [1]
    status: resolved
  - key: webster-gate-card-bundles-three-units
    rounds: [1]
    status: resolved
  - key: retire-card-deletes-accumulated-card-shas
    rounds: [1]
    status: resolved
  - key: pre-fix-head-missed-on-dirty-failure
    rounds: [1]
    status: resolved
  - key: loomcli-parent-branch-wiring-untested
    rounds: [1]
    status: resolved
  - key: resumed-merriam-and-webster-row-verify-unsettled
    rounds: [1]
    status: resolved
  - key: impactsummaries-restate-change
    rounds: [1]
    status: resolved
  - key: docs-card-misses-final-summary-and-verb
    rounds: [1]
    status: resolved
  - key: shell-readfile-left-dead
    rounds: [1]
    status: resolved
  - key: webster-gate-commit-check-needs-gitwrap-target
    rounds: [1, 2]
    status: resolved
  - key: gitwrap-integration-fix-leftovers
    rounds: [1, 2]
    status: resolved
  - key: merge-in-warning-promises-bisect
    rounds: [2]
    status: resolved
  - key: stale-bisect-comments
    rounds: [2]
    status: resolved
  - key: detach-primitives-outlive-caller
    rounds: [2]
    status: resolved
---

Round 1 reviewed a 17-card plan and raised eleven findings.
All eleven were fixed in a 19-card plan, with the old card 14 split into cards 14–16.
The judge then opened two findings about `internal/websterengine/gitwrap.go`.
Card 15's fixer-commit walk needs a `*gitrepo.Repo`, and only `gitwrap.go` may build one.
Card 17's deletion of `integrationfix.go` left gitwrap helpers and a `FabricBisector` comment stale.

Round 2's review restated both and added three stale-bisect findings: merge-in's warning and help text, three code comments, and the detached-checkout primitives D4 retires.
The fixer moved `gitwrap.go` into card 15's targets with two named helpers, widened card 17's targets to cover the leftovers and stale text, and added card 20 to retire `CheckoutDetached`/`RestoreBranch`.
The judge confirmed every fix against the repository and approved.
Nothing is open.
