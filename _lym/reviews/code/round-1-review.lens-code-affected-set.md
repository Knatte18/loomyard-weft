```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I found two stale or displaced comments and nothing else the diff leaves wrong.

**Per change, the set I found:**
- **`Result.Lacking`, `Module.Lacking` and the `lacking` envelope key renamed to `Unsettled` and `unsettled`:** Covered. No `result.Lacking`, `LackingTemplateBlocks` or `"lacking"` remains in Go, Markdown or YAML outside `_lym` and `.lym`. A repo-wide grep for `lacking` in other file types returned only unrelated hits (reportengine, fabricengine, landingshed, shuttleengine).
- **New `<module>.template-base` file under the board config dir:**
  - Covered: every enumerator of the board config dir (snapshot-and-restore in `internal/configcli/hubwide.go:244-259` and `internal/hubreconcile/walk.go:454-475`), the commit file lists via `ChangedFilesRel` (walk, `lym fabric reconcile`, default-layer apply), and the clone record at `internal/fabriccli/clone.go:66-68`.
  - Covered: the name has no `.yaml` suffix, so no module reader (`ConfigFile`) picks it up.
  - Not covered: the stale doc comments in finding F1.
- **Seed-only files now follow the template instead of never being filled:**
  - Covered: docs the record names (`runprofile/doc.go`, `hubreconcile/doc.go`, `configcli/doc.go` at lines 88-91, `configreg` `SeedOnly`, `configengine` `doc.go` and `layers.go`, `docs/overview.md:378`, both template headers, the `PATTERN.md` line and `PATTERN-config-layers.md`).
  - Covered: `PATTERN-sparse-hub-base` is unaffected, because `profiles.yaml` and `models.yaml` are seed-only full files, not sparse bases.
  - Not covered: the two stale spots in F1.
- **`light` drops `code_reviewer_fan: ""`:** Covered. `internal/battencli/runsettings_test.go`, `internal/runprofile/runprofile_test.go` (`TestTemplateLadder`) and the `runprofile/doc.go` sentence are updated. No other fixture or doc names `light` as solo.
- **`yamlengine.FollowTemplate` added:** Covered by `yamlengine/doc.go`. `docs/shared-libs/yamlengine.md` lists no sibling helpers (`RenameKeys` and `DropKeys` are absent there too), so there is no list to extend.
- **`notifyReinterpretations` split into `mailOrch`:** Covered. The `InstallNotifier` doc is updated and the only other caller is the new `notifyTemplateFollowed`.
- **PATTERN changes and commit trailers:** My directive carries no declared-changes list. The commits have no `Pattern-Departure:` trailer and need none.

## Findings

**F1 — LOW — consistency — `internal/configcli/hubwide.go:163`**
- `reconcileDefaultLayer`'s doc says "It seeds absent bases sparse and never fills a present one."
- This is now false for a seed-only board file: the same function fills it from its template (adds, updates or removes against the base).
- The record asks for docs to drop "never fills or prunes a present one".
- `internal/configcli/doc.go:92` ("It never fills a base …") sits right after the new seed-only paragraph and reads as contradicting it.
- Fix: say that non-seed-only bases are never filled and that a seed-only file follows its template. Scope the `doc.go:92` sentence to hub bases the same way.

**F2 — LOW — consistency — `internal/configsync/configsync_test.go` around lines 908-912 and 1091**
- The new `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` was inserted between the doc comment of `TestReconcileHubWideAt_MigratesLegacyFabricConfig` and its function.
- That comment ("// TestReconcileHubWideAt_MigratesLegacyFabricConfig pins … (see TestReconcileAll_HubWideCopy).") now runs straight into the new test's comment.
- `TestReconcileHubWideAt_MigratesLegacyFabricConfig` at line 1091 has no doc comment.
- Fix: move the legacy-migration comment block back directly above `TestReconcileHubWideAt_MigratesLegacyFabricConfig`, leaving the new test's own comment above the new test.

**Tool calls:**
1. `grep` in `docs/` for `FollowTemplate`, `DropKeys` and `RenameKeys`, plus a repo-wide `grep -l` for `lacking`: to check the yamlengine doc list and leftover `lacking` mentions outside `.go` and `.md`.
2. `grep` for `ConfigDir(`, `TemplateBaseFile` and `.template-base` in non-test Go: to find other readers of the board config dir.
3. Read `internal/fabriccli/fabric.go` 690-740: to see the `runReconcile` write and commit path around the `ChangedFilesRel` change.
4. Read `internal/configcli/hubwide.go` 155-195: to read the `reconcileDefaultLayer` doc comment that the manifest only showed as a diff hunk.
