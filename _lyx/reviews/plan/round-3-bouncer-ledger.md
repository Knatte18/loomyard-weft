---
round: 3
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
    status: resolved
  - key: setowners-pull-refusal-inputs-unnamed
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: setowners-push-transport-row-missing
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Rounds 1 and 2 raised twelve findings in total; round 3's review shows the last of them, the unreachable `TestSetOwners` rows of card 4, now reached through `BeforePush` and `Sleep`, so all resolve.
Round 3 raised two new MEDIUM consistency findings on the same card: the diverged and dirty pull refusals name no reaching input (F1), and the push-side transport branch has no row (F2).
Both fixes are local test-paragraph edits that the finding text fully specifies.
