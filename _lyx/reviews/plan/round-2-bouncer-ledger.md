---
round: 2
ledger:
  - key: push-retry-owner-guard-unlicensed
    rounds: [1]
    status: resolved
  - key: adoptcli-cli-cobra-deviation-unlisted
    rounds: [1]
    status: resolved
  - key: setowners-retry-delay-not-injectable
    rounds: [1]
    status: resolved
  - key: innerrun-owner-trigger-lost-at-call-boundary
    rounds: [1]
    status: resolved
  - key: resume-prime-claim-failure-undecided
    rounds: [1]
    status: resolved
  - key: pattern-short-line-drops-exception
    rounds: [1]
    status: resolved
  - key: machinename-leaf-docs-missing
    rounds: [1]
    status: resolved
  - key: unrecorded-pair-bypass-unstated
    rounds: [1]
    status: resolved
  - key: cross-package-symbols-bundled
    rounds: [1]
    status: resolved
  - key: adopt-step2-reword-departs-d5
    rounds: [1]
    status: resolved
  - key: childspawnerror-unreachable-row
    rounds: [1]
    status: resolved
  - key: setowners-test-rows-unreachable
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised eleven findings; round 2's review found each fix present and not regressed, so all are resolved.
Round 2 raised one new BLOCKING design finding (F1): card 4's `TestSetOwners` rows for the push-deferral refusal and the replay-loss re-read name no input that reaches those branches.
The proposed repair is a test hook on `OwnerRequest` run between the setter's commit and its first push.
