# Card 10 — shell-changedir

**Create:**
- `plan:internal/shell#posixShell.ChangeDir` -> `func (p posixShell) ChangeDir(dir string) string`
- `plan:internal/shell#pwshShell.ChangeDir` -> `func (p pwshShell) ChangeDir(dir string) string`

**Edit:**
- `internal/shell#Shell`
- `internal/shell/shell_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The shell seam gains `ChangeDir(dir string) string`, the standalone statement that changes the shell's current directory to `dir`, quoted through the dialect's `Quote`: `cd` in the POSIX dialect and `Set-Location` in pwsh.
It exists so the orch verb's hook command, a change of directory to the prime chained with the verb, is built through `internal/shell` and spells no shell syntax itself (`PATTERN-shell-mechanics-seam`).
`shell_test.go` gains `TestShell_ChangeDir` in the shape of its siblings, covering both dialects and a directory with a space and an apostrophe, and `cmd/lyx/testdata/test-budget.yaml` raises `internal/shell` by that one test.

**ImpactSummary:** One method added to the seam interface, implemented by its two dialects; no other type implements `Shell`.
