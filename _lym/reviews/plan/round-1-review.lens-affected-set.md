```yaml
lens: affected-set
outcome: reported
```

The affected-set lens traced what each card's change forces outside the card, across all five cards. Every set is covered except one comment that card 4 falsifies.

**Your list (per change: the set found, and the cards that cover it):**

1. **`code_reviewer_fan` goes from guard to none.**
   - Readers: `ConfigSpec` and its doc comment, and `docs/overview.md:583` ("all six closed to the local layer"). Also `pattern/PATTERN-config-layers.md`'s Guard bullet and the `wantLocalDenied` list in `TestModules_EveryTemplateKeyClassified`.
   - Tests: the key loop in `TestLoadConfig_Refuses`, the two darn rows of `TestPrepareRunSettings` that expect a refusal (`internal/configreg/runsettings_test.go`), and `TestModules_LocalLayerRefusals`.
   - Checked and not affected: `docs/overview.md:374` and `:626`, `internal/loomengine/config.go:8` and `:22`, `internal/configcli/doc.go:56`, and `internal/loomcli/wiring_test.go:913`, which seeds the worktree layer, not the local one. Also `configsync_test.go:2126-2133` (board layer) and the base-layer test in `configcli_integration_test.go:348-457`.
   - Covered by card 1.
2. **Five profile blocks in `internal/runprofile/template.yaml` gain `code_reviewer_fan: ""`.**
   - Readers: `internal/runprofile/doc.go`, `TestTemplateLadder`, `TestTemplateDocsProfile`, the `battencli/runsettings_test.go:44-46` row "no profiles file…", and the template row of `TestProfilesCheckLoad`.
   - Checked and not affected: the `configcli_test.go:1077-1091` rows build their own `light` blocks. `docs/overview.md:376` names the profiles but not their contents.
   - Covered by card 2.
3. **`goal-scope` and `board-fit` are deleted and `board-scope` is added.**
   - Readers: the template's fans and header, `TestConfigTemplate`, `config_test.go:68-87` in `TestLoad`, `discussiontable_test.go:413-415` and `:459`, and the `[:6]` cut at `configsync_test.go:2064`.
   - Checked: no stencil, `doc.go` or doc file names either lens.
   - Covered by card 3.
4. **`repo-invariants` takes `way-forward`'s place in `review-plan`.**
   - Readers: the template, `TestConfigTemplate`'s writer check at `config_test.go:311-319`, and the `config_test.go` file header at line 4 ("each writer fan holding its reviewer fan's lenses").
   - Card 4 covers everything except the header.
5. **`code-pattern-fit` is deleted.**
   - Readers: the template, `sets/csharp.yaml:53`, `TestConfigTemplate`, the remove row at `config_test.go:578`, and `docs/dotnet-hub.md:169`.
   - Covered by card 5.

**Findings:**

- **LOW · consistency · card 4 (`internal/lenses/config_test.go` file header):** card 4 does not update the file header's claim that every writer fan holds its reviewer fan's lenses.
  - After card 4, `writerFanGaps` admits `repo-invariants` as a reviewer-only lens, so the claim at `internal/lenses/config_test.go:4` is no longer true.
  - Card 4's Intent does not mention the header.
  - Fix: card 4 updates the header to name the reviewer-only exception, under the file's self glyph or within its existing edit of `config_test.go`.

**Tool calls:**
1. Grepped tests for `code_reviewer_fan` outside the files already read, plus doc mentions of `LocalDenied` and fan keys: to find readers of the reclassification the pre-read lacked.
2. Read `docs/overview.md:370-378` and the seeding call in `loomcli/wiring_test.go`: to check the profile and `run_settings` doc lines and the layer that test writes.
3. Grepped docs and comments for the other lens names and procedure-lens wording: to find doc text the lens changes falsify.
4. Grepped the loom template and overview for "closed", "guard" and "local" wording, and darn docs for the retired fan keys: to find other "closed to the local layer" text.
5. Listed the test files that touch run profiles or Seed-Child: to find other readers of the profile template.
6. Grepped those tests for code-bounce and local loom file assertions: to check whether any asserts an exact seeded local document from the template profiles. I did not open the assertion at `battencli/lifecycle_integration_test.go:1844`. Its failure message reads "want review_max_bounces: 2", so it likely seeds its own settings rather than a template profile.
