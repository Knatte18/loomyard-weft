```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what reads, asserts or describes it and check whether the diff leaves any of it wrong.

**Per change, the set found and whether it is covered:**

1. **`internal/ideconfig/template.yaml`: `open_files.beam` goes from `[README.md]` to `[]`.**
   - `ideconfig.LoadConfig` and `ConfigSpec` (`internal/ideconfig/config.go:52-90`) read it. The keys and categories are unchanged and `beam: []` still loads as a non-nil empty list, the same way `prime: []` and `task: []` do. Covered.
   - `ideengine.seed` (`internal/ideengine/spawn.go:128`) passes `ide.OpenFiles.Beam` to `filesToOpen`. An empty list returns nil, so `CodeLauncher` gets no files. Nothing else needed.
   - The `TemplateDefaults` and `LocalOverrideSetsRenderMarkdown` rows in `internal/ideconfig/config_test.go:30-43` are updated in the diff. Covered.
   - The `ide` key list in `internal/configreg/configreg_test.go:138` is unchanged because no key changed. Covered.
   - `internal/ideengine/spawn_hub_integration_test.go:328` sets its own `open_files.beam`, so it does not depend on the template default. Covered.
   - The hub's `_bolt/_lym/config/ide.yaml` (`open_files: beam: []`) now equals the template. That file lives in the records repo, not the diff. A sparse hub base keeps a template-equal unit until `hubreconcile`'s one-time strip, so nothing breaks. Not a diff subject.
   - PATTERN-config-layers, "Config reinterpretation" (`pattern/PATTERN-config-layers.md:114`): the diff changes a template default, not what a hub base file's value means. A hub that set `beam` keeps its value and meaning, and a hub that didn't inherits the new default. No `configreg.Reinterpretation` is needed. Covered.
   - `ide` is not `SeedOnly` (`internal/configreg/configreg_test.go:107`), so no `.template-base` handling applies. Covered.
2. **`internal/ideconfig/doc.go` and `docs/overview.md:568` prose.** Both now say the template opens nothing for any place. A search of non-Go files outside `_lym` and `.lym` found no stencil, spec, fixture or other doc stating the old default. The only `open_files` and `render_markdown` mentions are in `template.yaml` and `docs/overview.md`. Root `README.md` doesn't mention it. Covered.
3. **Commit message (98ea299d).** It has no `Pattern-Departure:` trailer, and its claims (template opens nothing for any place, a hub adds entries) match the diff. Covered.
4. **Declared PATTERN changes.** The directive declares none, so there is nothing to check. Covered.

**Findings:** none.

Tool calls:
1. Searched non-Go files and test fixtures across the repo (excluding `_lym`, `.lym` and `docs/research`) for text stating the old `README.md` beam default, and for any stencil, spec or fixture naming `open_files` or `render_markdown`. The manifest's earlier greps had searched only `internal`, `docs/overview.md` and `README.md`, not stencils, tools or fixtures.
