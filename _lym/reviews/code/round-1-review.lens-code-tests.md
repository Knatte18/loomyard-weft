```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that pins it, or report that none does.

**Behaviour → pinning test**

1. **The template's `open_files.beam` now loads as an empty list.** Pinned by `TestLoadConfig/TemplateDefaults` at `internal/ideconfig/config_test.go:31`, which expects `Beam: []string{}`.
   - With `README.md` put back in `internal/ideconfig/template.yaml:3`, `slices.Equal` (`config_test.go:69`) returns false and the row fails.
   - `slices.Equal` treats nil and an empty slice as equal, so the row does not tell `beam: []` apart from a missing key. Both resolve to "open nothing" in `filesToOpen` (`internal/ideengine/spawn.go:173-194`), so the behaviour is still pinned.
2. **A local override that sets only `render_markdown` leaves `beam` at the empty default.** Pinned by `TestLoadConfig/LocalOverrideSetsRenderMarkdown` at `config_test.go:40-42`. It is an existing row with its expectation updated, not a new test. It is not redundant: it pins the local layer, which `TemplateDefaults` does not reach.
3. **A beam window opens no file by default.** Pinned by row 1, because `seed` hands `ide.OpenFiles.Beam` straight to `filesToOpen` (`spawn.go:128`), and an empty list gives no files. No `ideengine` test relies on the old default: the only beam `open_files` case sets its config explicitly (`internal/ideengine/spawn_hub_integration_test.go:328`).
4. **Doc prose in `internal/ideconfig/doc.go:9-10` and `docs/overview.md:568`** is static prose and exempt. No test asserts it, and none should.

**Test economy**

- The diff adds no new test or row; it updates two existing rows.
- The redundancy report lists `TestLoadConfig` as unique.
- No assertion on static prose was added.

**Findings:** none.

Tool calls: none. The diff, `config_test.go`, `config.go`, `template.yaml`, `spawn.go` and the `open_files` grep over `internal/ideengine` were already in context.
