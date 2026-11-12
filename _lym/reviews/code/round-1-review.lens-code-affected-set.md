```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens over the advisor diff.

**Per change, the set found:**

- `buildSettings` gains an `advisorModel` parameter.
  - Callers are one production site, `claudeengine.go:224`, and every test call site. The diff updates all of them, in `settings_test.go`, `scope_test.go` and `hookcmd_integration_test.go`. Covered.
- `settingsDoc.AdvisorModel` (`omitempty`) is a new key in `settings.json`.
  - Golden fixture: `TestBuildSettings_MatchesGolden` builds with `""` and the key is omitted, so no golden edit is needed. Covered.
  - `buildOpenSettings` and `openSettingsDoc` serve opened sessions, which take no `Spec.Advisor`. Not affected.
  - The resume line reuses `settingsFileName` (`command.go:513`), so a revived session keeps the advisor, as `doc.go` claims. Covered.
  - Other `Spec` constructors (websterengine, sheddriver, treadleengine, sampler, landingshed) leave `Advisor` empty, so their launches stay unchanged. Covered.
- `Spec.Advisor` is a new field.
  - `spec.go` documents it. The CLI sets it.
  - Codex refuses it in `validateSpec` (`codexengine.go:69`), the sole call site at `:110`, before anything is written.
  - The `Requirement` mechanism is not used. The writer's summary justifies that, and the choice is not a defect. Covered.
- The `--advisor` flag, its usage text and the `Long` sentence are in `run.go`.
  - `TestRunCmd_ModelSpec` pins the flag reaching the spec.
  - Docs: `docs/overview.md`, `claudeengine/doc.go` and `codexengine/doc.go` all carry the change. Covered.
  - `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` lists flags only in example commands and has no flag inventory to update.
- PATTERN fit:
  - `PATTERN-shuttle-provider-seam`: the claude `advisorModel` key stays under `claudeengine`, and `spec.go` states the field as provider vocabulary. Covered.
  - `PATTERN-refusal-way-forward`: it covers webster, shed and loom refusals, not codex engine refusals. Even so, the codex message names a way forward, and the test row asserts `way forward`. Covered.
  - No declared PATTERN changes exist to land, and none were found.
- Commit messages: the commit message of `45afadf7` states the change. No `Pattern-Departure:` trailer is present or needed. Covered.
- Stencils: none changed, so no wording check applies.

**Findings:** none.

**Tool calls:** one grep wave in one Bash call. I used it to look for other docs, flag inventories and `ForkSubagents` or `Spec` consumers that mirror run flags, which my context lacked.
