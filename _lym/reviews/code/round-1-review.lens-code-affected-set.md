```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, per change, what the diff touches outside its own lines.

**Per-change list**

1. The `code_reviewer_model: "opus[high]"` value added to the five darn blocks of `internal/runprofile/template.yaml`: covered.
   - The consumers I found all read the key through the existing `loom` key path (`internal/loomengine/template.yaml:14`, `internal/configreg/keymove.go:29`).
   - `internal/loomengine/config_test.go:669` validates the spec format, and `opus[high]` is already a valid spec elsewhere in the repo.
   - No code consumer pins the old darn values. A search for the old `code_max_bounces: N, code_hard_max_bounces: M` strings found only the five changed test files (configcli, configsync, hubreconcile) and `template.yaml`.
2. Test fixtures that rebuild an "older template" by string replacement (`lightDarn`, `lightDarnSolo`, `solidDarn`, `solidDarnBase`, `solidDarnTuned`, `solidDarnFollowed`): covered.
   - They are updated in `internal/configcli/configcli_integration_test.go:393-397`, `internal/configsync/configsync_test.go:940-942` and `internal/hubreconcile/ensure_integration_test.go:1477-1482`.
   - The `lightSamplerReview` and `lightSamplerNoReview` constants (`configsync_test.go:943-944`) correctly stay unchanged, because the sampler block is unchanged.
3. `internal/battencli/runsettings_test.go:71`, the default darn profile's docs output: covered. It now expects `code_reviewer_model: opus[high]`, and the key sorts after `code_max_bounces`.
4. The `docs/overview.md` description of profiles (lines 525-531): not affected. It lists the starting profiles and the verbs, and says nothing about reviewer models.
5. The `internal/runprofile/doc.go` addition and the `template.yaml` header comment: covered.
   - Both claim that every darn block takes `opus[high]` and that pawl, sampler and tapestry take the hub's model. The template as left matches.
   - The two comments use different wording. The header says "an Opus model-spec" and `doc.go` says `opus[high]`. This is not a defect.
6. The PATTERN files: nothing to land.
   - The directive declares no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file.
   - I found no entry that this change touches. `PATTERN-config-layers` (meaning changes ship a reinterpretation) concerns a changed meaning of a key. Here only a seeded default changes, and the seed-only reconcile carries it.
7. Commit messages: covered.
   - The messages match the diff, and none uses a `Pattern-Departure:` trailer.
   - The first commit's claim that "tapestry, pawl and sampler keep the hub's Code-Review model" matches the template.
8. Stencils and specs: no stencil is changed. A search of `contracts` and `internal` for Code-Review/sonnet default descriptions found only the unchanged `loomengine/template.yaml` lines. They describe the hub default (`sonnet[medium] by default`), which stays true.

**Findings**

None.

**Tool calls (3 of 6)**
1. A grep over `*.md`, `*.go` and `*.yaml` (excluding `_lym` and `.lym`) for files naming profiles or the old round pairs, plus `code_reviewer_model` in `*.md`. My context lacked the consumers and docs outside the diff.
2. A grep for the old darn `code_max_bounces`/`code_hard_max_bounces` strings in `*.go` and `*.md`. My context lacked any fixture or test outside the diff that still pins them.
3. A grep of `docs/overview.md` and the specs for profile and Code-Review default prose, then a read of `docs/overview.md` lines 524-537. My context lacked the doc text on profiles.
