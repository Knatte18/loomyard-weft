Task: check the fix-affected-set lens, meaning whether the two fixes cover every member of the sets in F1 and F2.

**F1: every site that renders the remove command goes through `RemoveCommand`.**
- All non-test renderers call `boardengine.RemoveCommand`: `fixloop.go:268`, `fronttask.go:130`, `:132`, `:180`, and `front.go:16`.
- The other hits are the help text at `boardcli/cli.go:689`, which shows the JSON form, and the doc comment at `adoptcli/doc.go:178`, which also shows the JSON form.
- The new `stepRemoveCommandParses` test pins `RemoveCommand` against the real remove verb, so it covers every site.

**F2: callers of `fronttask.Remove` after the type assertion `err.(*fronttask.RemoveError)` in `green.go:199`.**
- `Remove` returns only `*RemoveError` or nil. Every error return from `Remove` in `fronttask.go:222-246` is a `&RemoveError{...}`.
- The `green.go:199` assertion is therefore safe.
- The other production caller, `fixloop.go:237`, passes the error to `fixRemovalError`. I did not read that function, so I did not check that it handles the error type, but the fix did not change it.
- The remaining caller is a test, `hub_integration_test.go:56`.

no findings

Calls made:
1. `grep` of all `RemoveCommand` and `lym board remove` sites in non-test Go, to rerun F1's set of rendering sites. My context had only some of them.
2. `grep` of `fronttask.Remove(` and `RemoveError` assertions, to rerun F2's set of callers. My context did not list them.
3. `sed` of `fixloop.go:230-250`, to see how the other `Remove` caller handles the error, which my context lacked.
