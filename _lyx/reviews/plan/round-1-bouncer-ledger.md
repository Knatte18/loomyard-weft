---
round: 1
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
    rounds: [1]
    status: open
  - key: gitwrap-integration-fix-leftovers
    rounds: [1]
    status: open
---

Round 1 reviewed a 17-card plan and raised eleven findings.
The fixer split the old card 14 into cards 14–16, which brought the plan to 19 cards, and addressed every finding.
The judge confirmed each fix against the current plan and the repository.

Two findings are new and still open.
Both concern `internal/websterengine/gitwrap.go`.
Card 15's fixer-commit check needs `parentMergeRejection`, whose `*gitrepo.Repo` argument only `gitrepo.New` builds.
`cmd/lyx/rawgitmutation_test.go` bars `gitrepo.New(` outside `gitwrap.go` in websterengine, so card 15 must target `gitwrap.go`.
That is MEDIUM, and it blocks.
Separately, card 17's deletion of `integrationfix.go` leaves `integrationFixHeadRefusal` with no caller, along with possibly `commitsBetween` and `commitParentCount`.
It also leaves a stale `FabricBisector` mention in `rawgitmutation_test.go`'s header.
That is a NIT.
