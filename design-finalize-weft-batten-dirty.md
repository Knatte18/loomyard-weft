# Finalize blocks on the parent weft's uncommitted batten status records

`finalize-weft-batten-dirty` · Note · bug, fabric, batten

Imported from [issue #501](https://github.com/Knatte18/loomyard/issues/501).

Seen (publish-recovery, 2026-10-10): after `lyx loom approve`, Finalize stopped blocked with "fabricengine: merge preconditions failed: worktree dirty".
The task pair and its weft were clean; the dirty tree was the prime's weft (`loomyard-weft`, `main-weft`), holding two uncommitted batten records: `_lyx/shed/publish-recovery/status.json` and `_lyx/shed/test-suite-weight/status.json`.
Batten writes these from the prime while runs are in flight and commits them later through weft sync, so any Finalize that runs between a write and its sync commit hits the precondition.

Workaround: the orchestrator ran `lyx fabric commit` in the prime, then `lyx loom resume` in the pair.

Expected: Finalize's merge precondition ignores, or commits first, the parent weft's own batten records (`_lyx/shed/*/status.json` written by batten runs), so a sibling run's status write never blocks a landing.
