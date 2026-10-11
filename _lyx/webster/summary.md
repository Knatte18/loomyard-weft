# Plan gaps: card insert, remove and restore verbs

Batch 01-begun-card-bound reported `done` at head `c826f686c060c5f912a97b185b15082cbbf0548f`.
It holds five cards, committed in order: `begun-card-bound`, `planparser-renumber`, `planparser-restore`, `loom-card-verbs` and `plan-stencil-rules`.

What was built, per the fork's reply:

- `websterengine.LastBegunCard`, the frozen-card bound read from the run record.
- In `planparser`: the plan scratch paths, the raw I/O seam, the journal, the insert and remove renumber, and `RestoreRenumber`.
- In `loomcli`: the three `lyx loom` verbs `insert-card`, `remove-card` and `restore-renumber`, with their refusal rows, the lease and the docs.
- The writer, rework and review stencils now describe the verbs and carry the paired reachability rule.

Deviations:

- `internal/planparser/restore.go` was created and holds `RestoreRenumber` and its sentinels, a file outside the declared file-ops.
- The fork added the sentinels `ErrRenumberPosition`, `ErrRenumberOnlyCard`, `ErrRenumberSlug` and `ErrRenumberIntent`, each wrapping `ErrRenumberInput`, so each input refusal gets its own way forward.
- The Use strings take lowercase argument tokens (`<position>`, `<slug>`, `<intent>`) because the CLI form test accepts nothing else.

`record-batch` returned only informational scope-outside-plan warnings for helper symbols in the touched files.
The plan-level verify gate runs after this turn ends.
