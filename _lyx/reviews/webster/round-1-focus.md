---
round: 1
focus:
  - "internal/configengine/edit.go: does the extracted pure function return the order $VISUAL, $EDITOR, `code --wait` on every OS, notepad on Windows, nano, vi, with `--wait` ahead of the file path and an env value still taken as one binary name?"
  - "internal/configengine/editor_test.go: does each case in card 15's test list exist, each run against fake lookups with no process spawned, and does the Windows-with-nano case assert notepad?"
  - "Card 15 grep check: do the three old `falls back to notepad` and `notepad or vi` sites (`edit.go`, `buildConfigLong`, configcli `doc.go`) and the new text in configengine `doc.go` all state the new order?"
  - "internal/configcli/menu.go against `git show main:internal/configcli/menu.go`: is the contract kept (numbered list of `configreg.Names()` marked `(configured)` or `(default)`, `q` exits 0, non-number or out-of-range exits 1, otherwise `editOne`) with the unused `*lyxcwd.Location` parameter dropped?"
  - "internal/configcli Command and runConfig: is `menu` registered beside `reconcile` with `cobra.NoArgs` and a Short, is the cwd, editor and sync resolution one shared helper rather than a copy, and does the hand-written RunE check `clihelp.ShouldAbort` first and record its exit with `clihelp.SetExit`?"
  - "Bare `lyx config` and the unknown-argument refusal: do they still resolve no cwd and keep their RunE unchanged, and does `lyx config menu bogus` refuse before the handler runs?"
  - "configcli_test.go: are the restored `TestMenu*` tests adapted to the new signature, and do `TestBareConfigListsModulesAndVerbs` and `TestConfigLong_MentionsEditorFallbackAndSet` carry the new assertions card 15 and card 16 list?"
  - "Docs in the same commit: do `docs/overview.md`'s two `lyx config` lines, configcli `doc.go` and `buildConfigLong` name `lyx config menu`, and do changed comments follow the repository's CLAUDE.md rule of one sentence per line with clause breaks?"
  - "Scope: does the diff touch anything beyond the two cards' files, such as a live `_lyx/config/` file or `cmd/lyx`?"
---

Seed focus from the plan's two cards.
The review range is the branch's own commits after the rejected PR head `87672129`.
