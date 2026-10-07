# card-batching: batches 01–07 done, batch 08 stuck on a fabric-reference audit finding

Batches 01 to 07 (tokencount-fork-report through rebaseline-tail) were already recorded `done` when this session started, and this session did not touch them.

This session began batch 08 (calibrate-template) and forked one implementer.
The fork ran the calibration with `-history ../card-batching-weft`, found the fit poor (median ratio 0.044, spread 4.450 against a limit of 3), made no change and no commit, and wrote a report with `status: OK` and no deviations.

`record-batch 08` refused with `batch_failed`: the fork's calibration command referenced the fabric repository, which the audit treats as a correctness finding.
`recover-batch 08` then refused with `needs_fresh`, since recovery cannot check that finding.
Per the failure ladder the run stops here without retrying either verb.

The refusal names a way forward for the operator: with no commit made and the worktree clean, `lyx webster accept-audit --batch 8` followed by `lyx webster recover-batch 8`; otherwise `lyx webster reset --to start` and `lyx webster run --fresh`.
Card 8 is allowed to end without a commit when the fit is poor, so the expected result of a re-run is the same: no template change.


## Audit warnings

These findings (fork-audit policy findings, and drift about a later card) were recorded as warnings and did not stop the run.

- audit warning (fabric-reference): accepted by accept-audit --batch, the batch made no commit and left a clean tree: ran a fabric-referencing command ("ls -d ../card-batching-weft 2>&1 | head; go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft 2>&1 | tail -60") that can rewrite run state — an implementer fork must never touch the fabric repo directly
- audit warning (fabric-reference): accepted by accept-audit --batch, the batch made no commit and left a clean tree: ran a fabric-referencing command ("cat internal/batcher/template.yaml; ls ../card-batching-weft 2>&1 | head; go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft 2>&1 | tail -60") that can rewrite run state — an implementer fork must never touch the fabric repo directly
