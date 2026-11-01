# Card 1 — shell-detached-forms

**Create:**
- `plan:internal/shell#posixShell.DiscardOutput` -> `func (posixShell) DiscardOutput(cmd string) string`
- `plan:internal/shell#pwshShell.DiscardOutput` -> `func (pwshShell) DiscardOutput(cmd string) string`
- `plan:internal/shell#posixShell.ExitSuccess` -> `func (posixShell) ExitSuccess() string`
- `plan:internal/shell#pwshShell.ExitSuccess` -> `func (pwshShell) ExitSuccess() string`
- `plan:internal/shell#posixShell.CommandFlag` -> `func (posixShell) CommandFlag() string`
- `plan:internal/shell#pwshShell.CommandFlag` -> `func (pwshShell) CommandFlag() string`
- `plan:internal/shell#posixShell.ChangeDirToWord` -> `func (posixShell) ChangeDirToWord(word string) string`
- `plan:internal/shell#pwshShell.ChangeDirToWord` -> `func (pwshShell) ChangeDirToWord(word string) string`
- `plan:internal/shell#TestShell_DiscardOutputAndCommandFlag` -> `func TestShell_DiscardOutputAndCommandFlag(t *testing.T)`

**Edit:**
- `internal/shell#Shell`
- `internal/shell#TestShell_Exit`
- `internal/shell#TestShell_ChangeDir`

**Intent:**
The button click line (card 9) and the confirmed line (card 8) must change to the run directory, discard the `lym reed button` verb's output and end with a forced zero status in reed's pane shell dialect, and the verb (card 8) must run a button's `run` text through the configured shell with that dialect's command flag (D7).
`internal/shell` is the one place a pane-shell string is built (`PATTERN-shell-guard-table`), so the four forms join the `Shell` interface there, with one implementation per dialect:

- `DiscardOutput(cmd)` returns `cmd` with its standard output and standard error discarded: `cmd >/dev/null 2>&1` in POSIX, `cmd *> $null` in pwsh.
- `ExitSuccess()` returns the statement that ends the shell with status zero: `exit 0` in both dialects.
- `CommandFlag()` returns the flag that makes the shell binary run its next argument as a command line: `-c` for POSIX shells, `-Command` for pwsh.
- `ChangeDirToWord(word)` returns the statement that changes the shell's current directory to the directory `word` names, `word` being already one quoted shell word, which this method does not quote again: `cd <word>` in POSIX, `Set-Location -LiteralPath <word>` in pwsh.
  The click line passes `#{@lym_button_dir}`, which the render pins already quoted in reed's dialect (card 6) and tmux substitutes at the click without expanding it again, since `run-shell` does not format-expand its `-c` start directory (card 9).

`Exit` stays as it is; the interface doc comment of each new method states its use, and the package header comment of `shell.go`, which lists every mechanic the interface covers, gains the four.
No other type implements `Shell` (only `posixShell` and `pwshShell` in this package), so no other implementer changes.

Tests, all `//go:build tier0` like the rest of `shell_test.go`:
`TestShell_Exit` gains the `ExitSuccess` assertion per dialect (`exit 0`), and the new `TestShell_DiscardOutputAndCommandFlag` pins, per dialect, the exact `DiscardOutput("lym x")` string and the `CommandFlag()` value.
`TestShell_ChangeDir` gains a row per dialect pinning `ChangeDirToWord("'/a b'")` as `cd '/a b'` and `Set-Location -LiteralPath '/a b'`, the word passed through unquoted.

**ImpactSummary:** four methods added to the `Shell` interface with its two in-package implementations; no caller changes, since no code outside `internal/shell` implements it.

**Commit:** 1: Add output discard, zero exit, command flag and a quoted-word directory change to the pane-shell seam
