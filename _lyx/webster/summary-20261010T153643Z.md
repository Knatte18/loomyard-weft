# Webster run stopped at batch 01-verify-log-per-run

No batch finished, so the run ended `stuck`.

The batch 01 fork returned after several interim stops and reported all four cards committed.
`record-batch 01` then failed the batch with `batch_failed`: the audit found that the fork ran a fabric-referencing command (`lyx fabric --help`, piped through `head`) inside a single compound shell line, and an implementer fork must never touch the fabric repo directly.
The failed report was archived as `_lyx/webster/reports/01-verify-log-per-run-20261010T152931Z.yaml`.

`recover-batch 01` refused with `needs_fresh`, because the finding is one recovery cannot check.
Its message names two ways forward: `lyx webster accept-audit --batch 1` followed by `recover-batch`, if every recorded command is read-only, or `lyx webster reset --to start` followed by `lyx webster run`.
Neither is a step Master takes, so the run stops here.
Batches 05 and 10 were not begun.

The fork's own report listed these deviations from the declared file-ops:
- `internal/testkit/envkit/envkit.go` gained `Landing.CommitParentRecords` among the seams whose nil is legal.
- `internal/pairteardown/claim.go` is a new file.
- `internal/landingshed/deps.go` gained the `CommitParentRecords` field.
- `internal/fabricengine/dirtiness.go`, `diff.go` and `mergeerrors.go` hold new dirty-path helpers.

The operator decides between accepting the audit and a fresh restart.
