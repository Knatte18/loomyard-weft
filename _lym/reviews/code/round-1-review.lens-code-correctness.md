```yaml
lens: code-correctness
outcome: reported
```

**Findings list (code-correctness): no findings.**

Each case below was traced against `languageconfig.AppendSkills` (`internal/languageconfig/config.go:146-161`).

- **`FixerTable`, `internal/landingshed/fixer.go:137`, with `LanguageSkills` nil:** the result is a fresh copy of `fixerSkills`: prose, code-quality, testing. The package-level `fixerSkills` is never written, because `AppendSkills` allocates a new slice and never writes `own`. Correct.
- **`FixerTable`, with `LanguageSkills = ["scribe:golang-build"]`:** the result is prose, code-quality, testing, golang-build. This matches the `TestFixerTable` expectation in both modes, `fixer_test.go:74`. Correct.
- **`FixerTable`, with a language list that repeats a base skill:** for `["scribe:testing", "scribe:golang-testing"]` the result is the base three plus golang-testing, with testing appearing once. This is the same dedup `darnengine`, `websterengine`, `driftengine` and `prsession` use through the same helper. Correct.
- **`FixerTable`, with an empty-string or nil element in `LanguageSkills`:** `AppendSkills` treats it as an ordinary string. The caller's trust boundary is unchanged by the diff, so this is not a regression.
- **`conflictSkills`, `internal/mergeresolve/spec.go:107`:** the spec is built with `AppendSkills(conflictSkills, deps.LanguageSkills)`. With `deps.LanguageSkills = ["scribe:golang-testing", "scribe:code-quality"]` the result is prose, code-quality, testing, golang-testing, so the duplicate code-quality is dropped. This matches the new third row in `mergeresolve_test.go:698`. The nil rows (`:696-697`) give prose, code-quality, testing. Correct.
- **Landingshed integration assertions, `finalize_integration_test.go:188` and `publish_integration_test.go:94`:** the expected list is the base three plus `scribe:golang-testing`, which is what `AppendSkills` produces for the conflict seat. Correct.
- **Sampler `skills`, `internal/samplerengine/spec.go:24` and `:96`:** the four-skill list is assigned directly, with no merge and no mutation. The shared package slice is passed by reference, exactly as before the diff, so the aliasing risk is unchanged. The list is fixed, so there is no dedup or ordering hazard. The `spec_test.go:45` expectation matches the code. Correct.
- **`FixerInputs.LanguageSkills` comment, `fixer.go:68`:** "after its own, before its prompt" matches the `AppendSkills` order. Correct.
- **`mergeresolve/doc.go:33`:** the skill order stated matches the code. Correct.

Tool calls:
1. `grep conflictSkills|AppendSkills internal/mergeresolve/*.go`: my context did not show how `conflictSkills` is consumed (that it goes through `AppendSkills` and is not used raw).
