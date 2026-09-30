# Webster survives a parent merge between a fork's commit and record-batch

All six batches finished `done`, and the integration suite reported `OK`.

- **01 head-reconcile-helpers:** `internal/websterengine/gitwrap.go` gained `reconcileReportHead`, a merge-only head walk, and `refuseMidMerge`, a mid-merge probe, with integration tests.
- **02 record-batch-tolerates-merge:** `record-batch` uses both helpers, records the batch at the report's own head, and the package doc states the rule.
- **03 recover-batch-tolerates-merge:** `recover-batch` follows the same rule. A moved HEAD adds a warning naming the merge SHA, and the doc covers both verbs.
- **04 parent-merge-wedge-scenario:** an end-to-end test covers begin, fork commit, parent merge, record and next begin.
- **05 run-in-flight-predicate:** `RunInFlight` is exported over webster's own `state.json` and `outcome.yaml`.
- **06 merge-in-warns-mid-run:** `lyx fabric merge-in` adds a `warnings` envelope key mid-run and never refuses. The `Long` help and the `ly-drive` skill document it.

Deviations: no fork reported any. Record-batch listed informational scope-outside-plan warnings for test helper symbols added in test files.

The integration verify (`go build`, unit tests and integration-tagged tests for websterengine, fabriccli and cmd/lyx) exited 0 at HEAD `af701526d`.
