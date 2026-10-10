# Raise the cmd/lyx test budget row to the merged count

Batch 12 (`cmd-lyx-budget-row-after-merge`) raised the `cmd/lyx` row in `cmd/lyx/testdata/test-budget.yaml` from 74 to 75, so the budget admits the merged package's top-level test count.
It landed as commit `1d1daf7f9cc4cccec899c5cc6be83a267c5cca1c`, which changes only that file.
`record-batch` returned `status: done` with no reported deviations and no warnings.
