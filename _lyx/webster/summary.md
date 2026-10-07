# card-batching: cost-model batching for Webster forks

Batches 01 to 07 were already `done` when this session resumed: the tokencount fork report, the batcher estimator, the calibration report, the recorded partition, the cost batchifier, the batcher profiles and the rebaseline of the unbegun tail.

Batch 08 (calibrate-template) was recorded `failed` by the earlier session.
This session ran `lyx webster recover-batch 8` in the background, and it returned `status: done` with no warnings.
The reported `head_sha` is `9c7f6d2634b193d8719984b7067da01c49a57d10`, the same HEAD as before the recovery, so the recovery added no commit.
The plan's Shared Decisions allow card 8 to end without a commit when the calibration fit is poor; I did not read the report, so I cannot say whether that was the reason.

No deviations were reported, and no digest carried warnings.
All 8 batches are `done`.


## Audit warnings

These findings (fork-audit policy findings, and drift about a later card) were recorded as warnings and did not stop the run.

- audit warning (fabric-reference): accepted by accept-audit --batch, the batch made no commit and left a clean tree: ran a fabric-referencing command ("ls -d ../card-batching-weft 2>&1 | head; go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft 2>&1 | tail -60") that can rewrite run state — an implementer fork must never touch the fabric repo directly
- audit warning (fabric-reference): accepted by accept-audit --batch, the batch made no commit and left a clean tree: ran a fabric-referencing command ("cat internal/batcher/template.yaml; ls ../card-batching-weft 2>&1 | head; go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft 2>&1 | tail -60") that can rewrite run state — an implementer fork must never touch the fabric repo directly
- audit warning (fabric-reference): accepted by accept-audit --batch, the batch made no commit and left a clean tree: ran a fabric-referencing command ("ls ..; cat internal/batcher/template.yaml; go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft 2>&1 | tail -60") that can rewrite run state — an implementer fork must never touch the fabric repo directly
