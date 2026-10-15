---
verdict: APPROVED
---

## Measure

Board entry `config-set-validate` (issue #620): `lyx config --set` validates the resulting layer the way the module's loader does, refuses a value its loader would reject naming the key, and at least refuses a list given to a single-model key instead of storing it as a quoted string.
The decision record carries no `Added after Discussion` entries and no carried-over review risks, so the board body is the measure unamended.

## Diff against the entry

PR 626 at `0ff3c4df7` against `main`:

- `internal/configreg/configreg.go` adds `Module.CheckLoad`, set on every registered module through two adapters over the loaders; `configreg_test.go` pins it non-nil for every module.
- `internal/configcli/configcli.go` routes the worktree and local `--set` writes through `setChecked`, which snapshots the file, writes, loads through the module's loader and rolls back on a new load failure; the error carries the loader's, which names the key.
- `internal/configcli/hubwide.go` adds the same load check after the strict check on the board-dir write.
- Tests cover the issue's exact case on the local, worktree and default layers, the restore of a present file, a per-round list still written as a list, and a write onto an already-failing stack.
- Docs: `internal/configcli/doc.go`, `lyx config --help` and `docs/overview.md`'s `--set` line.

Every point of the entry is met, and nothing in the diff goes beyond it.
The before/after rule, under which a stack already failing to load with the same error still takes a write, is a bound the entry does not rule out.

## Overlaps

`lyx loom peers` lists no parent commits; `own_row` is 22.

- `twill-fork0` (row 12): both change `docs/overview.md`, this run at the `lyx config --set` lines and that run at the twill lines.
  Text-only overlap at different places, no design conflict.
  Landing order: this run (row 22) is further along and lands first; `twill-fork0` takes the change at Merge-In.
- `shed-tackler` (row 6) and `tokencount-fixes` (row 0) list no changed paths, and their subjects touch no config `--set` behaviour; no overlap.

Board entries near this one, `loom-model-key-names` and `model-downgrade`, concern loom's model key names and role models, not `--set` validation; this run supersedes neither.
