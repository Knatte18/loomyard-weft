# Card 15 — `lyx ide spawn` seeds a marked VS Code keybindings block

**Create:**
- `internal/vscode/keybindings.go`
- `internal/vscode/keybindings_test.go`

**Edit:**
- `internal/vscode/config.go`
- `internal/ideengine/spawn.go`
- `internal/ideengine/testmain_test.go`
- `internal/idecli/cli.go`
- `internal/idecli/cli_test.go`
- `internal/idecli/testmain_test.go`
- `docs/overview.md`

**Intent:**
Make VS Code's integrated terminal forward the five Alt keys to tmux (D15 and the exclusive-temp-file addendum): on every run, `lyx ide spawn` writes a `// lyx:begin` … `// lyx:end` block into the user's VS Code `keybindings.json` holding one `workbench.action.terminal.sendSequence` entry per key, `"when": "terminalFocus"`: Alt+Up/Down/Right/Left as `\u001b[1;3A`, `B`, `C`, `D` and Alt+z as `\u001bz`.

`keybindings.go` (VS Code format knowledge only, no hub geometry):

- `UserKeybindingsPath() (string, error)`: `$XDG_CONFIG_HOME/Code/User/keybindings.json` (default `~/.config/Code/User`) on Linux, `~/Library/Application Support/Code/User` on macOS, `%APPDATA%\Code\User` on Windows; a pure inner form takes the GOOS, an env getter and the home directory so all three are unit-tested.
- A pure merge, `mergeKeybindings(existing []byte) ([]byte, error)`, over a JSONC scan that skips `//` and `/* */` comments and string contents, so a bracket or marker text inside a comment body or a string never matches; a marker counts only as a whole `// lyx:begin` or `// lyx:end` line comment.
  An existing block, accepted only as exactly one begin and one end marker, begin first, both inside the top-level array, is replaced in place, markers included.
  A file without one gets the block inserted on its own lines directly after the array's opening `[`, every block entry ending in a comma (VS Code accepts the trailing comma this can leave before `]`), so no byte outside the inserted lines changes.
  No top-level array, a begin without an end, an end without a begin, a repeated marker, an end before its begin, or a marker outside the array is a skip error naming the shape.
  Inserted lines use the file's own line ending (`\n` for a file without a line break), and the block ends with a line terminator after `// lyx:end`.
- `SeedKeybindings(path string) KeybindingsResult` performing the checks in order: a symlinked path is resolved and written at its target; a missing file is created as `[`, the block, `]`; a file that exists but cannot be read (permission denied, a directory in its place) is skipped; a merge error is skipped; a content-identical result writes nothing and reports unchanged, whatever the permission bits; a file whose permission bits deny the user write access is skipped before anything is written; otherwise the new content goes to a temporary file beside the target, named uniquely to the run and created exclusively (`O_CREATE|O_EXCL`), given the target's permission bits, and renamed over it.
  A failed write, a read-only directory or an existing file of the temporary name included, leaves the target untouched, removes the temporary file only if this run created it, and is a skip with its reason.
  `KeybindingsResult` carries the outcome (`created`, `written`, `unchanged`, `skipped`) and the reason.

Bound: the verb writes only between its own markers, plus the inserted block's own lines, in that one file, and the one temporary file beside it, which a run that completes never leaves behind; nothing outside the markers is read for meaning or changed.

`ideengine.Spawn` returns the keybindings result beside its error and seeds the block before launching; the path comes from a package-level seam `KeybindingsPath` (default `vscode.UserKeybindingsPath`), like `CodeLauncher`, and a failure to resolve it is a skip.
A skip never fails the spawn: `lyx ide spawn` still launches and reports the outcome and reason in its JSON result (`keybindings: {outcome, reason}`).
Both `testmain_test.go` files point `KeybindingsPath` at a temporary directory for the whole package run (process-global state, named in a comment), so no test writes the operator's real file.

Tests:
`keybindings_test.go` is one table over `mergeKeybindings`: insert with and without a trailing comma on the last entry, with comments and brackets inside strings and comments, replace an existing block in place, CRLF file kept CRLF, every malformed marker shape and a file with no array skipped; plus filesystem rows over `SeedKeybindings` in a temp dir: missing file created, second run unchanged (idempotent), permission bits kept, a read-only file already holding the block reported unchanged, a read-only file needing a change skipped and untouched, an unwritable directory skipped with no temporary file left, a directory in the file's place skipped, a symlink written at its target; and a `UserKeybindingsPath` table over the three GOOS values.
`idecli/cli_test.go`'s spawn dispatch row asserts the envelope carries the keybindings outcome and that the seeded file under the test path holds the block.

`config.go`'s package doc names the keybindings block; `docs/overview.md`'s ide line states that `spawn` seeds the block.

**ImpactSummary:** `ideengine.Spawn` gains a return value (one caller, `idecli`) and one user-file write bounded by its markers.
