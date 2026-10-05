# Card 15 — the editor fallback prefers code and nano over vi

**Edit:**
- `internal/configengine/edit.go`
- `internal/configengine/doc.go`
- `internal/configcli#buildConfigLong`
- `internal/configcli/configcli_test.go`
- `internal/configcli/doc.go`

**Create:**
- `internal/configengine/editor_test.go`

**Intent:**
With neither `$VISUAL` nor `$EDITOR` set, `configengine.DefaultEditor` opens vi, which the operator cannot use.
The resolution order becomes:

1. `$VISUAL`, then `$EDITOR`, as today.
2. `code --wait` when `code` is on PATH, on every OS.
3. On Windows, `notepad`.
4. Elsewhere, `nano` when it is on PATH, and `vi` only when neither `code` nor `nano` is.

Extract the choice into an unexported pure function in `edit.go` that takes the environment lookup, the PATH lookup and the OS name and returns the command as a binary plus its arguments, so `code` carries `--wait` ahead of the path.
`DefaultEditor` calls it with `os.Getenv`, `exec.LookPath` and `runtime.GOOS`, and runs the result with the file path appended.
The spawn and exit log lines name the resolved binary as today.
An environment value is still taken as one binary name, as today; splitting `$EDITOR` into arguments is out of scope.

Update `DefaultEditor`'s godoc, `configengine`'s `doc.go` (its `[Edit]` line names the order), `buildConfigLong` and `configcli`'s `doc.go` to state the same order.
Grep for `falls back to notepad` and `notepad or vi`: every match states the new order.

Tests, untagged, in the new internal test file `editor_test.go` (package `configengine`, since `edit_test.go` is the external `configengine_test` package), table-driven over fake lookups, with no process spawned:

- `$VISUAL` set wins over `$EDITOR` and over `code` on PATH.
- `$EDITOR` set, `$VISUAL` empty: the `$EDITOR` value, with no arguments.
- neither set, `code` on PATH, OS `linux`: `code --wait`.
- neither set, `code` on PATH, OS `windows`: `code --wait`.
- neither set, no `code`, OS `windows`: `notepad`, even with `nano` on PATH.
- neither set, no `code`, `nano` on PATH, OS `linux`: `nano`.
- neither set, neither on PATH, OS `linux`: `vi`.

`TestConfigLong_MentionsEditorFallbackAndSet` also asserts the Long names `code --wait` and `nano`.

**ImpactSummary:** which editor `lyx config <module>` opens when `$VISUAL` and `$EDITOR` are unset; nothing changes when either is set.

**Commit:** 15: the editor fallback prefers code and nano over vi
