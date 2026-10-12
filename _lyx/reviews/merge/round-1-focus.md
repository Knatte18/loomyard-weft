---
round: 1
focus:
  - "cmd/lyx/main.go and cmd/lyx/clitree_test.go: does the new root `report` command (reportcli) interact correctly with `shortcutcli.Mount` and the CLI-tree guards that now recognise shortcuts? Check the audience annotation, the help index and the shortcut-name collision guard against `report`."
  - "PATTERN-told-geometry.md now binds `reportengine`, and PATTERN-leaf-packages.md and PATTERN-sole-parsers.md gained `haltnote` rows: does the task's new `internal/shortcutcli` (embedded file, root mounting, any path or cwd derivation) follow the changed told-geometry and leaf-package rules, and does `pattern/PATTERN-cli-cobra.md`'s package-naming list still hold with `reportcli` and `shortcutcli` both present?"
  - "cmd/lyx/testdata/test-budget.yaml: both sides added rows and raised `cmd/lyx`; does the merged row for `cmd/lyx` still match the merged tree's top-level test count, and are the new `internal/shortcutcli`, `internal/reportcli` and `internal/reportengine` rows consistent with their packages?"
  - "docs/overview.md: do the module table, package tree and module bullets from both sides stay consistent, including the `lyx --help` module list text in main.go against the shortcut heading?"
  - "internal/loomcli: both sides touch the package (task's status strand size, parent's halt-note writing); does either side's change assume the other did not happen, for example a shared constant, layout test or doc.go claim?"
---

Seed focus drawn from the three overlapping files, the overlapping packages and the changed pattern files.
