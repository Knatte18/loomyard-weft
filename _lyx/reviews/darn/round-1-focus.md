---
round: 1
focus:
  - "Rollback in `internal/configcli`: does every refusal path (worktree, local, board-dir, `--layer default`) restore the exact prior bytes or remove a created file, and skip fabric sync and board commit?"
  - "The before/after load comparison: does the 'new failure text' rule refuse the board entry's case (a list given to `plan`) on every layer, and can a refusal be wrongly skipped or a valid write wrongly refused?"
  - "`CheckLoad` wiring in `configreg.Modules()`: does every row set it with the loader that matches its module, including loaders that take a module name?"
  - "Docs landing with the change: `doc.go`, `lyx config --help` (2000-byte cap) and `docs/overview.md` against the repo's docs rule; does the summary's claim about the shortened help sentence hold?"
  - "Test economy: do the new `TestDispatchLayers` rows duplicate existing rows, and is any added behavior (board-dir check, already-broken stack write) left uncovered?"
---

Seed focus from the board entry (`--set` must refuse a value the module's loader rejects, naming the key) and the change description.
