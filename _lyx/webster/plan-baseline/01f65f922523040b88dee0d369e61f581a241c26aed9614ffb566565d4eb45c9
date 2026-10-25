# Card 15 — landing-fix-attempts-key

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed#ConfigSpec`
- `internal/landingshed#LoadConfig`
- `internal/landingshed#retiredKeyRenames`
- `internal/landingshed#MigrateConfig`
- `internal/landingshed#MergeIn.align`
- `internal/landingshed/template.yaml`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/landingshed#TestLoadConfig_InvalidKeys`
- `internal/landingshed#TestMigrateConfig`
- `internal/landingshed_test#newMergeInDepsAt`

**Intent:** D11: `landing.yaml`'s `merge_fix_attempts` becomes `parent_align_fix_attempts`.
`Config.MergeFixAttempts` becomes `ParentAlignFixAttempts` with the new yaml tag and a doc naming Parent-Align; `ConfigSpec`'s category, `LoadConfig`'s at-least-one check and the template line follow, the template comment reading "fixer turns one Parent-Align row entry may spend".
`retiredKeyRenames` gains `{From: "merge_fix_attempts", To: "parent_align_fix_attempts"}`, so reconcile renames a hub's key with its value and reports it, a document already holding the new key keeps its value and drops the old one, and `configreg.PrepareRunSettings`, which runs the same `MigrateConfig`, migrates a run profile's or `run_settings`' old key at read time with a Warn; `MigrateConfig`'s doc names the rename.
`MergeIn.align` reads the renamed field, so the renamed value takes effect.
`TestMigrateConfig` gains a row whose document holds `merge_fix_attempts: 5`, wanting `parent_align_fix_attempts: 5` and the line `merge_fix_attempts: renamed to parent_align_fix_attempts`, and a row holding both, wanting the new key's value kept; the other tests and fixtures listed take the new name.

**ImpactSummary:** One landing.yaml key renamed; its readers are the Merge-In fixer cap and landing's config tests.
