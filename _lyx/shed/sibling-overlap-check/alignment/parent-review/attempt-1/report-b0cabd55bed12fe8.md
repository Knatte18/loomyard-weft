---
verdict: APPROVED
---

## Scope against the board entry

The decision record covers both parts the entry's Scope asks for, and nothing outside them.

- Warning on a new `depends_on` that makes a task a sibling of another task with a shared module label: D1 defines the sibling, D2 computes it in `boardengine`, and D3 names the sibling and the shared labels in the success envelope while the write still succeeds.
- Path-based overlap waits for note `labels-from-diff`: listed under Scope Out and Open risks.
- The sibling rule goes into the `ly:board` skill beside its `depends_on` guidance, with no orch-role copy: D5 and Scope Out.
- Batten starting one task per overlapping group stays with note `batten-ready-overlap`: Scope Out.
  D2's exported overlap function only makes reuse possible; it builds none of that note's behaviour.

D4 extends the warning from the entry's named `set-deps` and `upsert` to `upsert-batch` and `merge`.
Both write the same upsert fields, `depends_on` included, so leaving them silent would reopen the route the entry's Seen section describes.
The record labels this as an auto-mode assumption with that reason.
This is the entry's goal applied to every route that writes the field, not work belonging to another entry, so it is not a scope deviation.

No other board entry plans a board-write warning or a `ly:board` skill rule on siblings, so the design duplicates and contradicts none.
Notes `labels-from-diff` and `batten-ready-overlap` are referenced as future consumers and left untouched; `board-viewer` and `doctor` carry the `board` label but cover unrelated work.

## Overlap with other runs and parent commits

`lyx loom peers` lists no parent commits.
No peer's changed paths or records touch `internal/boardengine`, `internal/boardcli` or `plugins/ly/skills/board/SKILL.md`; `engine-registry`, which has no changed paths yet, plans no board work in its decision record.

The one shared file is the bookkeeping file `cmd/lyx/testdata/test-budget.yaml`, which this run may edit under its test-budget constraint and which `report-timing`, `viewer-off-shell` and `creel-core` change.
It is named here and does not escalate.
If it does overlap, each of those peers is further along (rows 12, 14 and 12 against this run's `own_row` 3), so the peer lands first and this run takes its changes at Merge-In.
