# Board labels and skills

All 14 batches reported `done`, in plan order, and each is committed.

## Config machinery

- Batch 01 (`yamlengine`): `Reconcile`, `MissingKeys`, `FillMissing` and `SetValues` take a trailing `openMaps ...string` and carry declared open maps whole.
- Batch 02 (`configengine`): `Load`, `LoadOrTemplate` and `Set` pass the open maps through to `yamlengine`.
- Batch 04 (`configreg`, `configsync`): `configreg.Module` gains `OpenMaps` and a `Lookup` function, and `ReconcileAll` passes the open maps to reconcile.
- Batch 05 (`configcli`): `lyx config --set labels.<name>=…` adds or rewrites one label, and it refuses a list-shaped `labels`.
- Batches 01, 02 and 05 add new `doc.go` files for `yamlengine`, `configengine` and `configcli`.

## Board

- Batch 03: `types` and `labels` in `board.yaml` load as ordered maps (`[]Label`), and the list shape still loads with empty descriptions.
  `ConfigOpenMaps` is declared, and the template drops `undecided`.
- Batch 06: `lyx board labels` prints both maps in file order.
  The new `boardcli/doc.go` and `SKILL.md` state the labels principle.
- Batch 07: `lyx board get --body` prints the body verbatim.
- Batch 08: `lyx board merge --body-file` sets the upsert body from a file or stdin.
- Batch 09: a merge carries the removed entries' issues.
- Batch 10: `maxSlugLength` is now `MaxSlugLength`.
- Batch 11: nine slug-taking verbs state the limit in their help, derived from the constant.

## CLI tree invariant

- Batch 12: bare `lyx config` lists, an unknown argument refuses, and the picker (`menu.go`) is gone.
- Batch 13: `lyx selfreport` refuses an unknown subcommand.
- Batch 14: `cliTreeFindings` and the known-finding path are deleted, so every group meets the invariant outright.

## Reported deviations from the declared file-ops

- Batch 11: `internal/boardcli/intake.go`, where the `import` help lives.
- Batch 12: `cmd/lyx/clitree_test.go`, `cmd/lyx/main_test.go` and `internal/configcli/configcli_integration_test.go`.
  Two of those were forced by the behaviour change: entries had to leave `cliTreeFindings`, and the `dispatch` signature changed.
  `main_test.go` now runs `config --print`.
- Batch 13: `cmd/lyx/clitree_test.go`, where `cliTreeFindings` was emptied.

Batches 09 and 11 reported a process slip: a test file was edited through a shell heredoc rather than `Edit`.
Both forks reported correct diffs and passing tests.

Every `record-batch` envelope carried only informational scope warnings.
The plan-level verify has not run yet.
