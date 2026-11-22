```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change in the diff, find what else calls, reads or asserts it, and report any of it left wrong.

**Changes and the affected set found**

1. **`samplerengine.SessionInputs.LanguageSkills` and `SessionSpec` setting `Spec.Skills`.**
   - Production callers: the one closure at `internal/loomcli/wiring.go:1362-1371`, which passes `languageSkills`. That variable is `language.Skills` from the single load at `wiring.go:653`, the same value the darn writer (`darnwiring.go:328` via `c.languageSkills`, set at `wiring.go:1325`), burler, webster and the gate fixer read. Covered.
   - Parity builder: `buildSampler` (`internal/shuttleparity/producers_test.go:231-254`) now sets `Skills` from `languageSkillsOf` (`seats.go:168`), the same way the other code-role builders do. Covered.
   - Consumers: `shuttleengine/run.go:596` and `shuttleparity/prepare.go:90` merge `Spec.Skills` after the role's own through `agentconfig.Config.Skills` (`config.go:248-260`). They need no change, and the new doc comment's "after the sampler role's agents.yaml skills" matches that merge order. Covered.
   - Tests that asserted no skills: `spec_test.go:61` is updated. A grep of the other sampler tests (`samplerwiring_test.go`, `loomshed/samplersession_test.go`) finds no assertion on Skills. Covered.
   - Package doc: `samplerengine/doc.go` gains a line on language skills. Covered.
   - Repository docs: `docs/overview.md:707` says skills come from `agents.yaml` "followed by `language.yaml`'s skills for a code role". `internal/languageconfig/config.go:15,39` and `docs/overview.md:352` say "every code role". The sampler is now one of those code roles, so nothing contradicts the change. The sampler section (`docs/overview.md:1133-1150`) says nothing about skills. Covered, no edit forced.
   - Stencil: the `sampler-template-agent` stencil does not mention skills, so no stencil wording is affected.
   - PATTERN: `PATTERN-engine-parity` needs every producer's spec compared on both engines. The producer entry sets the same Skills, so both manifests carry the skill loads. Covered. No other PATTERN entry is touched, and no PATTERN change is declared, so none had to land.

2. **`writeSamplerConfig` moved into `hubfixture_helper_test.go`.**
   - Callers: `TestWire_SamplerRecipe` (`wiring_darn_test.go`), which now calls the package-level helper with the same signature and body, and the new call in `TestWire_LanguageSkills`. Covered.

3. **The doc comment and body of `TestWire_LanguageSkills`.** No other file depends on them. Covered.

4. **Commit messages.**
   - The `Root-Cause:` trailer on 2f607a494 names the defect the diff removes (SessionSpec built no Skills and the wiring passed none).
   - No `Pattern-Departure:` trailer is present, and none is needed.
   - The changed assertion in `TestSessionSpec` (from "no skills" to "the given skills") follows the fasit's own fix ("change the test to pin that the skills are carried"). It is not a loosened check, so no `Loosened-Check:` trailer is needed.

**Findings:** none.

**Tool calls**
1. A grep of docs, stencils, Go files and fixtures for sampler and skills mentions, plus `RoleSampler` users: the context had no repo-wide search for other readers or asserters.
2. A grep of markdown for "language skills" and `language.yaml`, plus the sampler stencil: the context lacked the repository docs that describe which roles load language skills.
3. A read of `docs/overview.md:700-715` and a grep for "code role": to check the skill-loading rule doc against the new behaviour.
4. A read of `docs/overview.md:1133-1150`: the sampler recipe's doc section was not in the context.
