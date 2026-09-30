---
round: 1
ledger:
  - key: remove-go-header-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: removecase-doc-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: mutation-record-test-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: refusal-remedy-test-comments-fixed-column-wrap
    rounds: [1]
    status: resolved
  - key: refusalof-test-retired-anomaly-name
    rounds: [1]
    status: resolved
---

Round 1 reviewed the card-1 commit `015c710c2`, which reorders `Topology.Remove` so every refusal precedes the archive and the archive precedes every teardown.
It found no correctness defect in the reorder, the deleted remedy machinery, or the replacement tests.
The only findings were four LOW comment-convention violations, fixed-column wraps in the rewritten comments of `remove.go`, `livestate_verbs_test.go`, `mutation_record_integration_test.go` and `remove_refusal_remedy_integration_test.go`.
It also raised one NIT, a stale "Remove anomaly path" phrase in `refusalof_test.go`.
The fixer resolved all five in comment-only commits `8cdbdee18`..`1599d4717`, and the judge verified each fix plus the build and the integration Remove tests.
No entry is open.
