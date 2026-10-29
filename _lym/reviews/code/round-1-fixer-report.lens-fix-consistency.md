```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **F3:** The reworded Limitation in `internal/vscode/config.go` agrees with the lines above it, which already say `WorkspaceSettings` sets the look before the splice. The remaining "verbatim" in `internal/vscode/workspace.go:24` describes `BuildWorkspace`'s own contract for its input, so it is still correct. I found no other doc that calls the hub file's settings verbatim.
- **F2:** The new subtest matches `workspaceWindowTitle`, which returns `"<name> workspace"` when no shortname is recorded. It passes the repo name for the hub file and the slug for the beam file, as the code does. It uses the exported `fabricengine.ShortnameFileName` and `h.BoardDir()`, the same helper the sibling subtests use.
- **F1:** The `checkPrimeWriteFailure` doc comment was updated to cover the `Seed` assertions. The error substring `"write workspace file"` is the same one the `Spawn` half asserts, and the `Seed` half does not touch the launcher that half checks.

Tool calls (1 of 6):
- Grepped `verbatim` near workspace/settings across Go, `docs/`, `PATTERN.md` and `pattern/`, to look for a twin of the Limitation wording that my context lacked.
