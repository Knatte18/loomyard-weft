---
verdict: APPROVED
---

## Scope against the board entry

The board entry `config-set-round-list` asks that `lyx config --set` write a per-round model list as a list on `review`, `fix`, `fan_review` and each segment's `*_review` and `*_fix`, and refuse a value it cannot write in the key's shape rather than store it as a string.
The decision record carries no `Added after Discussion` entries, so the board body is the measure.

The diff at `552c8987e` against `main` meets both points:

- `internal/yamlengine/set.go` and `internal/yamlengine/layers.go` give `SetValues` and `SetSparse` a branch for a declared open-map path whose template value is a scalar or a list: a value starting with `[` is written as a flow list of scalars, any other value as a string.
- A bracketed value that does not parse as a list of scalars, or carries text after the closing bracket, is refused with an error naming the key and the quoted-entry form; nothing is written.
- `loomengine.ConfigOpenMaps` already lists `review`, `fix`, `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix`, `webster_review`, `webster_fix`, `fan_review` and `discussion_advisors`, which covers every per-round model key in `internal/loomengine/template.yaml`, so no loom change is needed.
- Docs move in the same change: `internal/yamlengine/doc.go`, `internal/configcli/doc.go`, `lyx config --help` and the config paragraph of `docs/overview.md`.

No change falls outside the entry; mapping-valued open maps such as board's `labels` keep their behaviour.

## Open risks carried over

The Darn-Review carry-over lists `open-map-list-shape-overwritten-by-set` (BLOCKING) and `trailing-text-after-bracket-dropped` (MEDIUM) as open in round 1.
Commit `0331e1931` addresses both: `scalarOrListOpenMapPaths` reads the template before `carryOpenMaps`, and `scalarOrListNode` refuses text after the closing bracket, with test rows for each.
Neither names an unrecorded parent correction.
The record's other open risk, a literal string starting with `[` at such a key now parsing as a list, touches no current key.

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commits; `own_row` is 22.

- `overview-seats` (row 12): both change `docs/overview.md`, at different places; this run adds one line to the `lyx config` paragraph.
  Text-only overlap, no design conflict.
  Landing order: this run (row 22) lands first, and `overview-seats` takes its change at Merge-In.
- `perch-budget-arbiter` (row 12): lists no changed paths; its decision record covers loom and darn review budget keys and the arbiter, not `yamlengine` or `--set`.
  No overlap.

No board entry is made obsolete by this run.
