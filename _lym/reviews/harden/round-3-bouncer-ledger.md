---
round: 3
ledger:
  - key: seed-board-push-lands-after-rollback
    rounds: [1, 2]
    status: resolved
  - key: faulted-fetch-refspec-read-as-two-repositories
    rounds: [1, 2]
    status: resolved
  - key: prime-key-names-missing-branch
    rounds: [1]
    status: resolved
  - key: seed-way-forward-misnames-hook-decline
    rounds: [1]
    status: resolved
  - key: seeded-records-branch-has-no-upstream
    rounds: [1]
    status: resolved
  - key: prime-key-refusal-spells-suffix-literal
    rounds: [2]
    status: resolved
---

Round 3 raised no finding and re-drove the round-2 fixes on a one-repository sandbox hub.

- `seed-board-push-lands-after-rollback` is resolved: a killed seed push now reads origin first, withdraws a landed ref, and keeps the hub when origin cannot answer.
- `faulted-fetch-refspec-read-as-two-repositories` is resolved: a store whose only mark is a `branch.<b>.merge` value under `refs/lym/` is refused with a way forward.
- `prime-key-refusal-spells-suffix-literal` is resolved: every refusal renders the suffix through `RecordsBranchName`, and no production file outside `internal/weftname` spells it.
- The three round-1 keys stay resolved.
