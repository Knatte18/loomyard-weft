# Board labels and skills — rework round 1

Both batches recorded `done`, with no `deviations` field in either digest.
Each digest carried only informational `scope-outside-plan` warnings for symbols the batch touched beyond its declared glyphs.

## 15-editor-fallback

Head `82b111622c7a549f521dd15a6b9cac998fcc3cd9`.
The editor resolution now tries `$VISUAL` and `$EDITOR`, then `code --wait`, then `nano`, then `vi`, with notepad kept as the Windows fallback.
The change is a new `resolveEditor` in `internal/configengine/edit.go`, called by `DefaultEditor`, with new tests in `internal/configengine/editor_test.go`.
The `configengine` and `configcli` `doc.go` files and the `lyx config` help text state the new order.

## 16-config-menu

Head `ed94a4b45325e51ba6cf451413b9b88e729e1a0c`.
The module picker is restored as `lyx config menu` in `internal/configcli/menu.go`, and bare `lyx config` keeps its cobra listing, which now includes `menu`.
Cwd and sync resolution moved out of `runConfig` into a shared `resolveReal`, which `runMenu` also uses.
The `TestMenu*` tests are restored against the new signature, and `TestConfigMenuRejectsArgument` is new.
The `lyx config` help, `internal/configcli/doc.go` and `docs/overview.md` name the subcommand.
The fork reported one use of `sed -i` on a test comment, against the conversation rules; build and tests passed afterwards.
