# Card 11 — the pane shell seam gains a fallback and an exit statement

**Create:**
- `internal/shell#posixShell.OrElse`
- `internal/shell#pwshShell.OrElse`
- `internal/shell#posixShell.Exit`
- `internal/shell#pwshShell.Exit`
- `internal/shell#TestShell_OrElse`
- `internal/shell#TestShell_Exit`

**Edit:**
- `internal/shell#Shell`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D12's launch line needs two pane-shell constructs no dialect offers yet, and `PATTERN-shell-mechanics-seam` keeps every pane-shell statement inside `internal/shell`.

- `Shell` gains `OrElse(first, second string) string`: a line that runs `second` only when `first` fails.
  Both dialects return `first + " || " + second`, since PowerShell 7, the pwsh tmux launches, has the same pipeline-chain operator.
  The doc comment says each operand must be one pipeline with no statement separator, because `;` binds looser than `||` in both dialects.
- `Shell` gains `Exit() string`: the statement that ends the pane's shell, `exit` in both dialects.
  Typed into a strand's launch script, which the pane's shell sources in its own scope, it ends the shell, so the pane goes dead under `remain-on-exit`.
- No other type implements `Shell`.

Tests, in `internal/shell/shell_test.go`, one per method as the file already does:

- `TestShell_OrElse`: for both dialects, `OrElse("a", "b")` is `a || b`, and `Chain(OrElse("a", "b"), "c")` is `a || b; c`.
- `TestShell_Exit`: both dialects return `exit`.

`cmd/lyx/testdata/test-budget.yaml`: raise `internal/shell` by two.

**ImpactSummary:** Adds two interface methods to `internal/shell`'s `Shell`, implemented by its only two types; no caller changes until card 13.

**Commit:** 11: The pane shell seam gains OrElse and Exit
