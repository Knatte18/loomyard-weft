```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that would fail if it broke.

**Per behaviour: pinning test**
- Landing fixer seat loads `scribe:prose`, `scribe:code-quality` and `scribe:testing` before `in.LanguageSkills`: `TestFixerTable`, `internal/landingshed/fixer_test.go:74`. The assertion is an exact `slices.Equal` on the full list. It runs inside the loop over both `FixerModeParentAlign` and `FixerModeFinalize`, so both modes are pinned.
- A language skill that repeats a base skill is dropped, and the package-level `fixerSkills` is not aliased: no fixer-level row, and none is needed. The behaviour belongs to `languageconfig.AppendSkills`, which the diff does not change. `internal/languageconfig/config_test.go:246` covers it ("empty language list" row, plus the `own` fixture). The darn, webster and drift seats pin the dedup through the same function (`internal/darnengine/spec_test.go:107-112`).
- Conflict session gains `scribe:testing` (`conflictSkills`): `TestBuildConflictSpec_SkillsAndParentDirective`, `internal/mergeresolve/mergeresolve_test.go:696-698`. All three rows changed: with parent, no parent, and language skills appended after the base list.
- The same behaviour end to end: the two landingshed integration assertions, `internal/landingshed/finalize_integration_test.go:188` and `internal/landingshed/publish_integration_test.go:94`. Both would fail if the skill were dropped.
- Sampler gains `scribe:code-quality` and `scribe:testing`: `TestSessionSpec`, `internal/samplerengine/spec_test.go:45`. It is an exact-list assertion, so order and content are both pinned.
- Doc comment changes (`internal/mergeresolve/doc.go`, `fixerSkills`, `skills`, `FixerInputs.LanguageSkills`): static prose, exempt.

**Findings**
- None. Every behaviour the diff changes has a failing row. I found no added test that duplicates coverage, and no stale assertion.
- Edge case: `newFixerInputs` sets `LanguageSkills` to one entry, so an empty `LanguageSkills` is not run at the fixer level. `AppendSkills`' empty-list row covers that case, so I raise no finding.

**Tool calls:** none. The manifest already held every file and assertion I needed.
