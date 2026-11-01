# Card 1 — shell-detached-forms

**Create:**
- `plan:internal/shell#posixShell.DiscardOutput` -> `func (posixShell) DiscardOutput(cmd string) string`
- `plan:internal/shell#pwshShell.DiscardOutput` -> `func (pwshShell) DiscardOutput(cmd string) string`
- `plan:internal/shell#posixShell.ExitSuccess` -> `func (posixShell) ExitSuccess() string`
- `plan:internal/shell#pwshShell.ExitSuccess` -> `func (pwshShell) ExitSuccess() string`
- `plan:internal/shell#posixShell.CommandFlag` -> `func (posixShell) CommandFlag() string`
- `plan:internal/shell#pwshShell.CommandFlag` -> `func (pwshShell) CommandFlag() string`
- `plan:internal/shell#TestShell_DiscardOutputAndCommandFlag` -> `func TestShell_DiscardOutputAndCommandFlag(t *testing.T)`

**Edit:**
- `internal/shell#Shell`
- `internal/shell#TestShell_Exit`

**Intent:**
The button click line (card 9) and the confirmed line (card 8) must discard the `lym reed button` verb's output and end with a forced zero status in reed's pane shell dialect, and the verb (card 8) must run a button's `run` text through the configured shell with that dialect's command flag (D7).
`internal/shell` is the one place a pane-shell string is built (`PATTERN-shell-guard-table`), so the three forms join the `Shell` interface there, with one implementation per dialect:

- `DiscardOutput(cmd)` returns `cmd` with its standard output and standard error discarded: `cmd >/dev/null 2>&1` in POSIX, `cmd *> $null` in pwsh.
- `ExitSuccess()` returns the statement that ends the shell with status zero: `exit 0` in both dialects.
- `CommandFlag()` returns the flag that makes the shell binary run its next argument as a command line: `-c` for POSIX shells, `-Command` for pwsh.

`Exit` stays as it is; the interface doc comment of each new method states its use, and the package header comment of `shell.go`, which lists every mechanic the interface covers, gains the three.
No other type implements `Shell` (only `posixShell` and `pwshShell` in this package), so no other implementer changes.

Tests, both `//go:build tier0` like the rest of `shell_test.go`:
`TestShell_Exit` gains the `ExitSuccess` assertion per dialect (`exit 0`), and the new `TestShell_DiscardOutputAndCommandFlag` pins, per dialect, the exact `DiscardOutput("lym x")` string and the `CommandFlag()` value.

**ImpactSummary:** three methods added to the `Shell` interface with its two in-package implementations; no caller changes, since no code outside `internal/shell` implements it.

**Commit:** 1: Add output discard, zero exit and command flag to the pane-shell seam
