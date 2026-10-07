# card-batching: batches 01–07 done, batch 08 stuck on an audit finding

Batches 01 through 07 (tokencount fork report, batcher estimator, calibration report, recorded partition, cost batchifier, batcher profiles, rebaseline tail) were already recorded `done` when this session started, so this session built nothing new from them.

Batch 08 (`calibrate-template`) was begun and forked in this session.
The fork reported `status: OK` with `head_sha` equal to batch 07's HEAD and no deviations.
Its reply said it made no commit because the calibration fit was poor: 165 cards over 14 runs, with a spread of 4.450 against a limit of 3.
The plan allows card 8 to end without a commit in that case, so `internal/batcher/template.yaml` is unchanged.

`record-batch 8` then refused with `batch_failed`.
The audit found that the fork ran `go run ./tools/tokencount ... -history ../card-batching-weft`, a command that references the fabric repo, which an implementer fork must never touch.
The report was archived.

`recover-batch 8` refused with `needs_fresh`, a finding recovery cannot check.
The failure ladder ends the run there.
The refusal's own way forward is, since the batch made no commit and the worktree is clean, `lyx webster accept-audit --batch 8` and then `lyx webster recover-batch 8`.
Otherwise it is `lyx webster reset --to start` and `lyx webster run --fresh`.
Webster's checks did not reject any batch content; the stop is the audit finding alone.
