# cli-surface: batch 01 done, batch 06 stuck

Batch 01 (audience-and-index-renderer) recorded `done` at head `1666f9302cc84301d142cfbc1340a96dd99a1911` with no deviations.
It covers cards 1 to 5: the `audience` annotation and index renderer, `internal/helpcli`, the shed-verb audiences, and the small-module, reed and orch form rewrites.
`record-batch` listed only informational scope-outside-plan warnings for helper symbols in `clihelp`, `helpcli` and `shedverbs`.

Batch 06 (fabric-form) was implemented by its fork, which reported cards 6 to 11 committed with head `5819a6f5c48d809e3c4e308ed9c5fcc751f2d91c`.
`record-batch 06` then failed with `batch_failed`: the audit found that the fork ran a loop of `lyx fabric <verb> --help --json` commands, which webster treats as a fabric-referencing command that can rewrite run state.
The report was archived to `_lyx/webster/reports/06-fabric-form-20261010T144105Z.yaml`.

`recover-batch 06` did not run a recovery.
It returned an error from its own audit, which tried to resolve the offending command text as a path (`lstat .../fabric-reference: ... file name too long`).
No failure-ladder rung covers a retry of that error, so the run stops here.
Batch 12 was not begun.
The run is resumable with `lyx webster run` once an operator has looked at the batch 06 audit and the recovery error.
