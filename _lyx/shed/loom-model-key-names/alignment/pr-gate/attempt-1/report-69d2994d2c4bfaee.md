---
verdict: APPROVED
---

## Alignment at PR-Gate — `loom-model-key-names`, PR 708 at `f25fa7143`

The diff against `main` (`8023e9b69`) carries out the board entry's operator decision of 2026-10-17, and the decision record adds no `Added after Discussion` entry to measure against.

- Flat role names: `loom.yaml`'s `review` and `fix` become `reviewer_model` and `fixer_model`, the shared fallback an empty segment key takes.
  `<segment>_review` and `<segment>_fix` become `<segment>_reviewer_model` and `<segment>_fixer_model` for Discussion, Plan and Webster.
- Every review segment has its own reviewer and fixer key:
  Merge gains `merge_reviewer_model` and `merge_fixer_model`, wired to the Merge rows in `internal/loomcli/wiring.go` in place of Webster's.
  Darn gains `darn_reviewer_model` and `darn_fixer_model` in `darn.yaml`, resolved in `wireDarnReview` (`internal/loomcli/darnwiring.go`).
- Fan keys renamed in the same style: `<segment>_fan` and `<segment>_fix_fan` become `<segment>_reviewer_fan` and `<segment>_fixer_fan` for Discussion, Plan, Webster and Merge, and `darn.yaml`'s `review_fan` and `review_fix_fan` become `darn_reviewer_fan` and `darn_fixer_fan`.
- `lyx config reconcile` rewrites the old keys: `loomengine.MigrateConfig` renames them through the new `yamlengine.RenameKeys` and copies a set Webster value onto an unset Merge key.
  `darnengine.MigrateConfig`, registered in `internal/configreg`, renames the two Darn fan keys.
  `internal/configreg/runsettings_test.go` covers a task's `run_settings` naming a retired key.

No retired key name survives outside the migration tables, their doc comments, `docs/overview.md`'s reconcile paragraph and `docs/research/`.
The remaining `merge_review` hits are the Merge-Review rubric's stencil marker, an unrelated name.
The other role model keys and the non-model `review_*` knobs keep their names, as the entry's "more role models sit beside them" leaves them.

The record's open risks are the writer's own: a hub that overrode `webster_review` moves Darn-Review to `darn.yaml`'s default, which reconcile's report line names, and `run_settings` keeps old names on disk.
Neither names an unrecorded parent correction, and neither leaves a Done-when point open.

## Overlap

- `pr-gate-run-identity` (peer, row 9, Plan-Burler-Review): both change `internal/loomcli`, this run in `wiring.go`'s review-row maps and `darnwiring.go`, the peer in the landing wiring's pull-request lookup and `alignmentwiring.go`'s snapshot.
  They change different behaviour and no shared decision or PATTERN entry, so the overlap is text-only.
  Landing order: this run (`own_row` 22) is further along and lands first; the peer takes its changes at Merge-In.
- Parent commits: `lyx loom peers` lists none.
- Bookkeeping: the diff touches `cmd/lyx/testdata/test-budget.yaml`.

## Board

`shedrecipe-capabilities` depends on this entry and is not made obsolete by it, so no slug is superseded.
