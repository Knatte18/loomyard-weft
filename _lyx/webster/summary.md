# card-batching: batches 01-07 done, batch 08 stuck

Batches 01 to 07 recorded `done`, each with no deviations from the declared file-ops; the warnings were informational scope notes only.

- 01 tokencount-fork-report (`f28990732`): `tools/tokencount` lists each Webster fork with its cards, messages, peak context and weight.
- 02 batcher-estimator (`5132cd2d7`): `internal/batcher` gains the size source, profile weights parse and segment-cost function; `planparser.RefFile` maps a ref to a file.
- 03 calibration-report (`3c9604e41`): the calibration report and two new `internal/gitrepo` reads.
  The first fork died on an API error without a report, and the re-fork finished the work.
- 04 recorded-partition (`830066308`): `Batcher.Batch` takes the plan, cards and a size source; webster records the partition at first init, every verb reads it, and batches run under an order assertion.
- 05 cost-batchifier (`097e23793`): hard limits and an exact DP over contiguous segments; the plan spec drops `Batchable?`.
- 06 batcher-profiles (`427fca8b8`): named profiles and `active:` resolution, load errors naming `batcher.yaml`, template profiles.
- 07 rebaseline-tail (`9c7f6d263`): rebaseline keeps begun batches and re-batches the unbegun tail.

Batch 08 (calibrate-template) made no commit: its fork found the calibration fit poor, which the plan allows.
`record-batch 8` nevertheless returned `batch_failed`, because the fork ran a calibration command that named `../card-batching-weft`, the fabric repo, which the audit forbids.
`recover-batch 8` refused with `needs_fresh`, so the run stops here.
The way forward from the refusal is `lyx webster reset --to start`, then `lyx webster run --fresh`.
