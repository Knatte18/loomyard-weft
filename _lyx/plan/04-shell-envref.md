# Card 4 — shell-envref

**Edit:**
- `internal/shell/shell.go`
- `internal/shell/posix.go`
- `internal/shell/pwsh.go`
- `internal/shell/shell_test.go`

**Intent:**
Add `EnvRef(key string) string` to the `Shell` interface: the shell syntax that expands the named environment variable, read at the time the pane shell runs the line, into exactly one argument.
POSIX returns `"${KEY}"` (double-quoted, so an unset or spaced value stays one word);
pwsh returns `"$env:KEY"`.
The Claude engine needs it to pass `--name` as a reference to `$LYX_STRAND_NAME`, which reed exports only after shuttle has built the command line (Shell Mechanics Seam).

`shell_test.go` gains a table case per dialect asserting the exact string.

**ImpactSummary:** Adds one method to the stdlib-only Shell interface; its two implementations are the only ones, so no other implementer breaks.

**Commit:** 4: shell: EnvRef expands an env var as one argument
