```yaml
lens: fix-consistency
outcome: reported
```

No findings that reach MEDIUM or above. One LOW.

**LOW — array-form association exception missing from the sibling docs**
- Changed text: `applyMarkdownPreview` in `internal/vscode/config.go`, which now leaves a non-map `workbench.editorAssociations` untouched.
- Stale siblings:
  - `internal/vscode/config.go:68` (the `WriteConfig` doc) says "With renderMarkdown on, either file's `*.md` editor association is set to the markdown preview". That no longer holds for the array form.
  - `internal/vscode/config.go:11` (the package doc) says "a seed asked to render markdown points that entry at the markdown preview". It omits the exception too.
  - `docs/overview.md:408` says a seed points the entry at the markdown preview with no qualifier.
- Evidence: the new early return `if present && !isMap { return }` runs before the render-on path sets the entry, and the new `RenderKeepsArrayFormAssociations` row pins that the array is kept.
- The `applyMarkdownPreview` doc itself was updated, so only its callers' docs disagree with it.

Checked and consistent:
- The `PATTERN-config-layers.md` degrading list now matches `configStrictnessDegradingSet` (`internal/ideconfig` is at `cmd/lym/configstrictness_test.go:63`).
- The `seed` and `OpenTask` doc comments both say they read over slug's own worktree's config stack.
- The `ideconfig` docs at `internal/ideconfig/doc.go:10` and `config.go:50` agree with the local-override behaviour.
- `overview.md:406` says "an entry naming no file skipped", which covers the directory-skip row.

Tool calls:
1. Grepped `internal/ideconfig`, `internal/ideengine`, `internal/idecli`, `docs` for config-stack wording and `internal/vscode`, `docs` for association wording. My context did not hold the sibling docs.
2. Read `docs/overview.md` lines 404–412 to see the ide section around the association sentence. My context did not hold it.
