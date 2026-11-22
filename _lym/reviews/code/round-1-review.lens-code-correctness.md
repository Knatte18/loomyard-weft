```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches.

**Per function or branch: the case run, and the result**

- `wiring.go:1369`, the sampler closure passing `LanguageSkills: languageSkills`:
  - Case: language.yaml lists `[scribe:golang-comments, scribe:golang-testing]`. `languageconfig.Load` sets `languageSkills` at `wiring.go:653` before any recipe branch. The closure captures that one variable, the same one darn (through `c.languageSkills`, `wiring.go:1325`), webster (`:768`, `:878`) and the fixers read. **Correct.**
  - Case: `skills: []` gives an empty or nil slice, so the spec carries no extra skills. **Correct.**
  - Case: an unusable language.yaml makes `wire` return at `:651` before the sampler branch, so no spec is built from a half-loaded value. **Correct.**
  - Case: the closure runs lazily, after `wire` returns. `languageSkills` is a local that nothing reassigns after `:653`, so the value it captures does not change. **Correct.**
- `samplerengine.SessionSpec`, the new `Skills: in.LanguageSkills`:
  - Case: two Go skills are carried onto `Spec.Skills` unchanged. **Correct.**
  - Case: nil gives nil Skills. `run.go:595` still expands the role's agents.yaml skills through `ConfigRole` (unchanged), and `len(spec.Skills) > 0` decides whether the skill loader runs. **Correct.**
  - Case: every required-field or engine error returns before the literal, so the change touches no error path. **Correct.**
  - Aliasing: `agentconfig.Config.Skills` (`config.go:254`) appends onto `slices.Clone(own)`, so the caller's `languageSkills` backing array is never written. Many specs share that one slice without risk. **Correct.**
- The `LanguageSkills` doc comment's ordering claim ("after the sampler role's agents.yaml skills"): `config.go:254` merges role skills first, then the extra ones, deduplicated. That matches the claim. **Correct.**
- The new sampler block in `TestWire_LanguageSkills`:
  - Case "listed language": the sampler spec's Skills equal the chair's Skills, and the chair's Skills equal `wantSkills`. So the sampler is pinned to the literal list. **Correct.**
  - Case "skills: []": `slices.Equal(nil, []string{})` is true, so the check passes whether the result is nil or empty. Both mean "none", so this is acceptable. **Correct.**
  - Error rows return before the sampler block. That is consistent with the sampler's own error path (`:651`), which is the shared load. **Correct.**
- `buildSampler` in `shuttleparity`: a `languageSkillsOf` error returns `Spec{}` and the error, and the caller at `producers_test.go:512` fails on it. Otherwise it sets `Skills` the way production sets it. **Correct.**
- `writeSamplerConfig`, moved into `hubfixture_helper_test.go`: same body and signature as the old closure in `TestWire_SamplerRecipe` (the parameter was renamed from `hubPath` to `hub`), so behavior is unchanged. **Correct.**

**Findings**

None.

**Tool calls**

1. `sed -n 640,660p internal/loomcli/wiring.go` and `sed -n 585,605p internal/shuttleengine/run.go`: the manifest held line 653 only as a grep hit, not the error path around it. It also lacked the shuttle merge site's surrounding control flow, which I needed for the nil-Skills and loader branches.
