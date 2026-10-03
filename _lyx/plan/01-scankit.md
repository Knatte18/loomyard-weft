# Card 1 — scankit

**Create:**
- `internal/testkit/scankit/scankit.go`
- `internal/testkit/scankit/scankit_test.go`

**Intent:** Add the shared source-scan harness (D1) as a new kit, `internal/testkit/scankit`, importing the standard library only.
It provides the API named in the overview's `scankit-api` Shared Decision: the module root from the kit's own source location via `runtime.Caller` and the enclosing `go.mod` (no process spawned), a walk over named roots with the shared skip-dir set, a production/test/all file filter, an extension filter, a shallow (package-directory-only) mode and slash-normalized relative paths, yielding each file's path, bytes and a lazily parsed `*ast.File`;
an allowlist type built from one-entry-per-line `{key, why}` pairs that reports both violations not on the list and stale entries that matched nothing;
a vacuous-scan floor;
and an import-allowlist assertion for one package directory (stdlib plus named import paths, parsed `ImportsOnly`).
Keep the decision logic in pure helpers that return values (stale keys, a floor error, import violations) so the unit tests drive them without a fake `testing.TB`; the `Require*`/`Assert*` wrappers only report through `t`.
The package doc states the kit's purpose and that it imports the standard library only.
`scankit_test.go` covers the walk filter (prod/test/all, extension, shallow, skip-dir set, slash paths, over a `t.TempDir()` tree), stale-entry detection, the floor, and the import-allowlist check over a temp package, including its negative cases.
Nothing converts to the harness in this card.
