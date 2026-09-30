# Card 1 — Shell: source statement and script extension

**Edit:**
- `internal/shell#Shell`
- `internal/shell/posix.go`
- `internal/shell/pwsh.go`
- `internal/shell/shell_test.go`

**Intent:**
Add the two dialect mechanics reed needs to type a short launch line instead of the full composed one, so reed emits no raw dialect syntax (Shell Mechanics Seam).

Add two methods to the `Shell` interface, each with a doc comment in the interface's existing style:

- `Source(path string) string` — a standalone statement that runs the statements in the file at `path` in the current shell's own scope, so exports and `PATH` changes land in the calling session.
- `ScriptExt() string` — the file extension, leading dot included, a script this dialect sources carries.

Implement both on `posixShell` in `posix.go`:
`Source` returns `". " + p.Quote(path)` (POSIX `. '<path>'`, with the `'\''` escape `Quote` already applies), and `ScriptExt` returns `".sh"`.

Implement both on `pwshShell` in `pwsh.go`:
`Source` dot-sources a script block built from the file's text, `". ([scriptblock]::Create(" + p.ReadFile(path) + "))"`, which yields `. ([scriptblock]::Create((Get-Content -Raw '<path>')))`.
It reuses the dialect's own `ReadFile` rather than spelling `Get-Content` a second time.
The doc comment records why it is never a dot-source of the file path: evaluating text runs no script file, so no ExecutionPolicy (`Restricted`, `AllSigned`, Windows PowerShell 5.1's client default) can refuse it, and a refused launch line would fail the strand launch.
`ScriptExt` returns `".ps1"`.

Tests in `shell_test.go`, in the file's existing table style, one test per dialect for each method:

- `Source` for a plain path, a path with a space, and a path with a single quote, asserting the exact string (`. '/a/it'\''s.sh'` on POSIX; `. ([scriptblock]::Create((Get-Content -Raw 'C:\a\it''s.ps1')))` on pwsh).
  The pwsh cases use Windows-style paths.
- The pwsh `Source` test also asserts the result equals `". ([scriptblock]::Create(" + Pwsh().ReadFile(path) + "))"` and does not start with `". '"`, pinning that it is the script-block form over `ReadFile`, never a dot-source of the file path.
- `ScriptExt` returns `.sh` for `Posix()` and `.ps1` for `Pwsh()`.

Update the `shell.go` file header and the `shell_test.go` file header, which both enumerate the mechanics, to name the new ones.
`internal/shell` stays stdlib-only.

**ImpactSummary:** Adds two methods to `Shell`; its only implementers are the two dialect types in this package, and every consumer (claudeengine, loomcli, reedengine) only calls it, so nothing outside `internal/shell` changes.

**Commit:** 1: shell: add Source and ScriptExt dialect mechanics
