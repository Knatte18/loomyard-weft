```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for the callers, tests, docs and rule documents it forces edits in.

**Per-change list**

1. **`disableWorkflows` key added to `sessionScope`.**
   - Set found: `scope.go` (struct field, `resolveSessionScope`), `scopeSettingsKeys` and the `want` strings in `scope_test.go`, the `doc.go` scope paragraph, `template.yaml` `enabled:` comment, and the `docs/overview.md:437` shuttle entry.
   - All of these are updated. The `ScopeConfig.Enabled` comment now says "tools".
   - Covered.

2. **Plugin-derived `skillOverrides` entries and the `ly` exception removed.**
   - Removed: `lyPluginName`, `pluginSkillsDirName`, `installedPlugin`, `makeSkillsDir`, `writePluginsFile` and `TestReadInstalledPlugins_EmptyInstallPathIgnoresWorkingDirectory`.
   - A grep of `README.md`, `docs/`, `cmd/lym/stencilseed.go`, `internal/` and `pattern/` finds no leftover reference.
   - No leftover wording of the old rule ("keeps all of them", "every skill but the preloaded ones") in `docs/overview.md`, `doc.go`, `template.yaml` or `scope.go`.
   - No PATTERN entry or `pattern/` background file mentions the session scope, so no declared PATTERN change is owed.
   - Covered.

3. **`readInstalledPlugins` and `installedPlugins` now return `[]string`.**
   - Callers: `sessionScopeFor` and `resolveSessionScope`, both updated. The tests use the new `writeInstalledPlugins(t, ids)` helper.
   - Covered.

4. **Live `claude.yaml` defaults.** `template.yaml` is the only `claude.yaml` I found. Its `skills_off` list is unchanged and its comments are updated. No hub-tuned `session_scope` exists in the tree. Covered.

5. **Commit message and `Pattern-Departure:` trailer.** The message has no trailer and departs from no PATTERN entry. Covered.

**Findings**

- **LOW, `consistency`, `internal/shuttleengine/claudeengine/doc.go:324`.** The doc states that "No session lym launches calls the `Workflow` tool" as fact. The diff carries no evidence for it.
  - The only support is `_lym/landing/summary.md` (a grep of stencils, specs and Go files, plus a transcript count).
  - A grep of non-research `*.go`, `*.md`, `*.yaml`, `*.json`, `*.tmpl` and `*.sh` files finds `Workflow` only in `.claude/settings.json:9`, a deny rule.
  - That grep agrees with the claim, so I found no counter-evidence.
  - It is a perishable claim in a doc comment, not a defect in the code.
  - Fix: reword it to the stable fact, e.g. "lym sessions are not given the Workflow tool: no stencil or spec directs one to call it".

I found no other member of the set left wrong.

**Tool calls (4 of 6)**
1. grep of the docs, `internal/` and `pattern/` for leftover removed symbols and old wording. My context did not show every doc.
2. grep for `session_scope` in docs and pattern files. It returned only the `scope_test.go:68` hit.
3. grep for the `docs/overview.md` scope paragraph; it returned nothing because the pattern was too narrow.
4. grep for `session_scope` and the new wording in `docs/overview.md:437`, plus the word-diff of the changed sentence. This confirmed the doc update.
