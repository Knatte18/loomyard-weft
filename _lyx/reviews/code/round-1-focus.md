---
round: 1
focus:
  - "Search all Go source, docs, stencils, contracts, scripts and `tools/sandbox/` for any remaining `lyx attach`, `lyx pause`, `lyx resume` or `lyx start` named as a command (acceptance criterion 4): does any survive, including in the command index and plugin skill files?"
  - "Refusals, notices and help text that named `lyx resume`/`lyx pause`/`lyx start`: do they name `lyx tender resume`/`lyx tender pause`/`lyx loom start` with the same arguments (D1), and does each refusal's test assert the new text and reach the way forward?"
  - "`StartAliasCommand` removal in `internal/loomcli/start.go`, `cmd/lyx/main.go` and the sandbox coverage allowlist: are the registration, its test and the comments naming it all gone, does `newLoomCLI` keep one caller, and does `lyx loom start` keep its flags including hidden `--no-attach`?"
  - "`internal/shortcutcli/shortcuts.yaml` and `shortcut_test.go`: does the file keep exactly `a`, `p`, `prime`, `r`, do the tests pin that set, and do the tender help text and `docs/overview.md` name `lyx p` and `lyx r` (D3)?"
  - "Test economy: do the edited and deleted tests in `internal/loomcli/cli_test.go` leave any behavior uncovered, and are any new assertions duplicates of existing coverage?"
---

Mostly a rename-by-text diff across many packages; the risk is a missed site or a message whose text and test drifted apart.
