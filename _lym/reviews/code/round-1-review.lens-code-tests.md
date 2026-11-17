```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per behaviour, the test that pins it:**

1. **`prOpen` refusal on a handed-over push must not say "again"** (`internal/beamcli/pr_test.go`, conflict resolution in `bae0ec63`).
   - Pinned by the handed-over row of `TestPROpen` at `pr_test.go:209-214`, through `wantNoErr: []string{"again"}`.
   - The loop at `pr_test.go:301-305` checks every `wantNoErr` substring on every refusal row.
   - No assertion was lost. On the beam side this row had only `wantNot` and never asserted "untouched". On HEAD it still checks `wantErr` and the "again" exclusion.
   - Leaving `wantUntouched` unset makes the loop return at `pr_test.go:306-307`, exactly as beam's version did.
   - Main's rows keep `wantUntouched: true` at lines 137, 143, 153, 171, 177 and 187. The `"lym beam merge-parent"` exclusion stays at line 152. The loop applies both fields, so no row asserts less than it did on its own side.
   - No `wantNot` is left anywhere in `internal/beamcli/` (grep found none).
2. **`internal/lymdirs/doc.go` package doc**: prose only. Exempt.
3. **`internal/reedengine/bindings_integration_test.go`, three waits using `recorderDeadline`**: test-only code with no production behaviour change.
   - `recorderDeadline = 30 * time.Second` (`bindings_integration_test.go:412`) is the same value as main's literals.
   - The waits at lines 162, 393 and the `ButtonClick` wait keep their predicates and messages.
   - `go vet ./internal/reedengine` passes.
4. **`382a591f`: comment rewrites in `internal/heddleengine/overview.go` (file header, `overviewPlace` doc) and the `TestRenderOverview` doc in `overview_test.go`**: comments only. No behaviour, so no test row is due.
   - The comment lint over the range at HEAD passes (`lym loom lint-comments --base 0c013afa…` returned `ok: true`).

**Findings:** none.

Tool calls: none beyond the inherited context.
