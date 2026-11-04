```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found:**

- `vscode.Launch(target, files...)` and `ideengine.CodeLauncher`:
  - Production callers are `Spawn` and `OpenTask` in `spawn.go`.
  - Test stubs are in `menu_test.go`, `spawn_hub_integration_test.go` and `idecli/cli_test.go`.
  - All callers and stubs are updated to the new signature. Covered.
- `vscode.WriteConfig(..., renderMarkdown, lymPath)`:
  - The only production caller is `writeVSCodeConfig`, reached through `writeWorktreeConfig`.
  - Test callers are the `config_test.go` call sites.
  - All are updated. Covered.
- `ide.yaml` as a new hub-wide config module:
  - `configreg.Modules`, `configreg_test.go` (flags and `wantLocalKeys`) and `configstrictness_test.go` (degrading set) are updated.
  - `docs/overview.md` gets a package-tree line and a spawn paragraph.
  - `pattern/PATTERN-config-layers.md` is not updated. Finding 1.
- `Seed` and `OpenTask` now fail when `ide.yaml` fails to load:
  - Callers are `idecli/cli.go` lines 77, 106 and 111, which return the error.
  - The spawn doc comments are updated.
  - The `PATTERN-config-layers` Strictness section's degrading-callers list is the other place that describes this behaviour. See finding 1.
- Declared PATTERN changes: the directive declares none, so there is nothing to check.
- Commit messages: no `Pattern-Departure:` trailer is present or needed.
- Stencils: the diff changes none.
- Other `desktop.yaml` docs: `docs/overview.md` is the only markdown file that names it, and it is already covered.

**Findings:**

1. **MEDIUM, class `consistency`.** `pattern/PATTERN-config-layers.md:59` lists the degrading callers as `{shuttleengine, ..., hibernateconfig, envconfig}`.
   - The diff adds `ideconfig` as a `configengine.LoadOrTemplate` caller and pins it in `cmd/lym/configstrictness_test.go:63`.
   - The background file's list is left without `ideconfig`, so the rule document no longer matches the pinned set.
   - Fix: add `ideconfig` to that list.
   - Evidence: `pattern/PATTERN-config-layers.md:59`, `cmd/lym/configstrictness_test.go:46-63`, `internal/ideconfig/config.go:158`.
   - `PATTERN.md:21` has no caller list, so it needs no change.

No other member of the affected set is left wrong.

**Tool calls (2 of 6):**
1. A grep for test callers of `CodeLauncher`, `vscode.Launch` and `WriteConfig` outside `config_test.go`, plus markdown files naming `desktop.yaml`. My context held the production callers but not every test caller.
2. The grep for the `hibernate` references earlier in the parent's context already covered the other config-module registries, so I made no further call.
