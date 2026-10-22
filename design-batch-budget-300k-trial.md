# Trial: batcher profile wide (budget 300k)

`batch-budget-300k-trial` · Note · audit, webster

## Seen

`run-process-ownership` (2026-10-22) ran mostly one card per fork under the `cautious` profile's budget 200k: its cards estimated 145-260k alone.
Measured fork peaks matched the estimates overall; `batch_growth` (7k) is above the ~2k measured when batches go well.
Many small forks were expensive under the identity batcher.

## Decision (operator, 2026-10-22)

`cautious` stays as it is; the hub's `batcher.yaml` adds a profile `wide`, the same as `cautious` but with `budget: 300000`, and makes it `active`.
The hub layer also still holds a copy of `cautious` at template values, left over from a first attempt (issue #833); the operator removes it in the editor.

## Follow-up

After a few loom runs, compare cost and fork quality against earlier runs; then decide whether `wide` goes into `internal/batcher/template.yaml`.
