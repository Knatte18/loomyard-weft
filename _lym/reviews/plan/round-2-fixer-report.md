# Fixer report — plan round 2

## Changes

- **F1 (BLOCKING), fixed as proposed.** Card 2 (`_lym/plan/02-buttons-config-keys.md`):
  - Create gains `plan:internal/reedengine#ConfigOpenMaps`.
  - Edit gains `internal/configreg#Modules` and `internal/configreg#TestLookup`.
  - Uses gains `internal/configreg/doc.go`.
  - The Intent's opening sentence no longer says "no load ever fails on a button's content". It now says reed declares its three groups as open maps, so a group of any shape loads, while `buttons` itself must stay a mapping.
  - The reed bullets add three things. `ConfigOpenMaps` returns `buttons.task`, `buttons.beam` and `buttons.prime`, and `ConfigSpec` sets `OpenMaps: ConfigOpenMaps()`. They state why: `yamlengine.MissingKeys` would otherwise require the template's list-element keys. The reed row of `Modules` gains the same open maps, which `TestModules_SeedOnlyAndHubWideFlags` compares.
  - `TestLoadConfig_TemplateDefaultsResolve` gains a row: a worktree file `buttons: {task: [{label: x, run: y}]}` loads, its `task` group replaces the default whole, and `beam` keeps the template's default. The rows for `buttons: null`, the mapping group, and the local-layer refusal are kept.
  - `TestLookup` gains a `reed` row whose `wantOpenMaps` is the three paths.
  - The ImpactSummary names reed's spec and registry row gaining three open maps.
- **F2 (LOW), fixed as proposed.**
  - Card 2 moves `docs/overview.md` from Extends to Edit.
  - Card 9 (`_lym/plan/09-button-click-binding.md`) moves `internal/toolfloor/doc.go` from Extends to Edit. Card 9's Extends section was left empty, so it was removed.
- **F3 (NIT), fixed as proposed.** Card 5 (`_lym/plan/05-third-status-line.md`) gains one Intent sentence: the header comment of `internal/reedengine/bar_integration_test.go`, which says it expands the two-line status bar, reads three-line. The file was not added as a target.

`lym loom validate-plan` returns `{"ok":true}` after the changes.

## Self-check

No lens returned a finding.

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings. It reran the searches for F1, F2 and F3, and none was left unchecked.

## Disputed

None.

## Deferred

None.
