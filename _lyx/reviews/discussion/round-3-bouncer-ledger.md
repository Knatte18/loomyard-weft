---
round: 3
ledger:
  - key: d1-deadline-bound-outcome-done
    rounds: [1]
    status: resolved
  - key: d6-environment-set-misses-unix-tag
    rounds: [1]
    status: resolved
  - key: d3-contract-file-bound-false-premise
    rounds: [1]
    status: resolved
  - key: d3-reset-own-path-set-unbounded
    rounds: [1]
    status: resolved
  - key: d2-reprompt-text-cannot-carry-findings-path
    rounds: [1]
    status: resolved
  - key: scope-item-5-probe-history
    rounds: [1]
    status: resolved
  - key: d6-behaviors-without-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: d6-rename-partitioned-declaration-underived
    rounds: [2]
    status: resolved
  - key: d6-base-context-builtin-tags-fail-open
    rounds: [2]
    status: resolved
  - key: d6-free-tag-cap-has-no-value
    rounds: [2]
    status: resolved
  - key: d3-recovery-contract-refusal-says-fork
    rounds: [2]
    status: resolved
  - key: d6-cost-claim-mismatches-matchfile
    rounds: [2]
    status: resolved
  - key: d6-rename-old-way-forward-per-file-create
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
---

Rounds 1 and 2 raised twelve findings between them.
Round 3 cleared the four round 2 findings it tested by focus verdict, and did not re-raise the fifth, so all are resolved.

Round 3 raised one new finding, F1 (MEDIUM, `design`): D6's partitioned Rename-old way forward says to Create the new member in each of its files, which the plan format refuses as `handle-collision`.
It is first raised in round 3 and stays open for Go to carry into Open risks.
