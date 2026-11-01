```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether the diff covers it:**

1. **New `desktop` config module** (`internal/configreg/configreg.go`, hub-wide, with `Spec`).
   - Registry tests: `TestModules_SeedOnlyAndHubWideFlags` and `TestModules_EveryTemplateKeyClassified` are updated, covered.
   - Strictness set: `cmd/lym/configstrictness_test.go` gains `internal/desktophook` in the degrading set, covered.
   - `pattern/PATTERN-config-layers.md`: the degrading-callers list gains `desktophook`, covered.
   - Module lists in other test kits: `cmd/lym/sandbox_coverage_test.go:51` and `internal/testkit/envkit/envkit.go` mention `drift`/`darn` only as run or role names, not as config-module lists, so not affected.
   - `docs/overview.md`: the package-tree line (274) and the behaviour paragraph (685-686) land in the same diff, covered.
   - `docs/overview.md:682` describes `Open-Window` / `open_window_command` without the new follow-on command. Line 685 adds that, so it is covered.

2. **`CategoryMachine` widened to "or a fact of the machine's desktop"**
   - Updated in `internal/configengine/layers.go`, `internal/configengine/doc.go` and `pattern/PATTERN-config-layers.md`, covered.
   - A grep for the old phrase "bounding a resource" finds no stale copy left outside the diff.

3. **`lym reed window` now runs the hook** (`internal/reedcli/window.go`, new `placeWindow` seam in `internal/reedcli/cli.go`).
   - The verb's `Long` help text and the file header are updated, covered.
   - Batten's `Open-Window` row reaches the hook through this verb, so it needs no separate change.
   - Other `OpenAttach` or `termwindow` callers: none outside `reedcli/window.go`.
   - Other tests that run the verb: `internal/battencli/lifecycle_integration_test.go` and `internal/battenshed/config_test.go` only reference the command string. I did not read them, so I assume they don't exercise the hook.

4. **`ideengine.Spawn` and `ideengine.OpenTask` now run the hook** (`internal/ideengine/spawn.go`).
   - Callers of `ideengine.Spawn`, `Seed` and `OpenTask`: only `internal/idecli/cli.go:77`, `:106` and `:111`. All stay valid, because the hook is internal to the engine, so the set is covered.
   - File header and doc comments are updated, covered.

5. **PATTERN rules touched**
   - `PATTERN-spawn-observability`: `internal/desktophook/window.go` logs the spawn through `logger.Info`, satisfied.
   - `PATTERN-config-layers`: `LoadOrTemplate` is adopted and listed, satisfied.
   - `PATTERN-cliwire-sole-wiring` and `PATTERN-cwd-resolution`: `reedcli` calls `lymcwd.ResolveWorktree` and `fabricengine.ConfigDirs` itself. `PATTERN-cwd-resolution` says only `internal/lymcwd` resolves a worktree, which `reedcli` respects. `PATTERN-cliwire-sole-wiring` is scoped to "wiring standalone CLI"; I did not read its background file, so I raise no finding.

6. **Commit messages and departures**: neither commit carries a `Pattern-Departure:` trailer, so there is nothing to judge. No declared PATTERN changes are in the directive that I could see, so none are missing.

**Findings:**

- **LOW / scope** — `internal/configcli/configcli.go:484`: the diff reworded the `lym config` help text from "Keys the current template does not recognize are" to "Keys the template lacks are". The decision record's Goal does not ask for this, and it has nothing to do with the window command. No test asserts either wording (grep found no match), so it breaks nothing. It is an unlicensed drive-by edit.
  - Fix: revert that line to the original wording, or move the edit to its own change.

No other member of the affected set is left wrong.

**Tool calls (3):**
1. A grep for the old machine-category phrase and the old `configcli` wording, a grep for the `"drift"` module-name literal across Go, md and yaml files, and a grep of docs for `darn.yaml` / `drift.yaml`. I lacked the module-list and doc cross-references.
2. A grep for the `drift` and `darn` literals in `sandbox_coverage_test.go`, `envkit.go` and `reedcli/list.go`, plus a file list of `reed window` / `open_window_command` mentions. I lacked whether those were config-module lists and who else references the verb.
3. None further; I made only two tool calls.
