```yaml
lens: fix-claims
outcome: reported
```

Findings:

- **LOW** — D6 third bullet, the rewritten text "so fabric names no workspace key", followed by "The beam file's generated mark is fabric's own, so fabric refuses a member named as that mark the same way."
  The bullet says fabric names no workspace key, and two lines later has fabric name and refuse one. Evidence: `internal/fabricengine/hubworkspace.go:42` declares `beamWorkspaceMark = "lym.generated"`, and `hubworkspace.go:227-229` writes it as the first top-level key of a beam file. The text should read "names no workspace key but its own generated mark". The wording came from the reviewer's proposed fix.

Every other added or rewritten claim holds at HEAD:
- **D5 `--board-path`:** the flag bypasses resolution and config, per `internal/boardcli/cli.go:134-136`. Without it, the board directory comes from the resolved layout, per `cli.go:153` and `161`.
- **D5 cwd:** the "anchored directory or its records sibling" scope matches `lymcwd.ResolveWithin` in `internal/lymcwd/lymcwd.go:135-163`.
- **D6 splice:** `vscode.BuildWorkspace` writes `folders` and then `settings`, per `internal/vscode/workspace.go:46-51`, so "after `settings`" and "keys it writes itself (`folders` or `settings`)" match. The beam mark is added by fabric after the build, per `hubworkspace.go:208`.
- **D1 escapes:** Go's `encoding/json` HTML escaping writes `&lt;`, `&gt;` and `&amp;` as a backslash, `u` and `003c`, `003e` or `0026`, which matches the new wording.
- **D6 presentation:** `"reveal": "always"` with `"focus": false` is a valid VS Code task presentation, and it shows the output without relying on how a silent reveal behaves on exit.

Tool calls:
- Read `internal/boardcli/cli.go:115-164` to check the `--board-path` branch. My context held only grep lines from it.
- Read `internal/fabricengine/hubworkspace.go:139-238` to check where fabric builds the files and adds the mark. My context held only grep lines.
- Read `internal/vscode/workspace.go:20-64` to check the key order `BuildWorkspace` writes. My context held only its first lines.
