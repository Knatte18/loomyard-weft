```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. For each change the diff makes, the set it forces outside its own lines, and whether the diff covers it.**

1. **`FixerTable` skills (`internal/landingshed/fixer.go`).**
   - Set found:
     - `fixerSkills` and its comment.
     - The `FixerInputs.LanguageSkills` comment, which was updated.
     - `TestFixerTable`, which asserts the full list in both modes.
     - Callers `parentalign.go:97`, `finalize.go:124` and `fixerrun.go:113`, which only pass `LanguageSkills` through, so no signature changed.
     - `landingshed/doc.go`, which mentions only the Describe session's skills and says nothing about the fixer's.
     - `landing-template-fixer` stencil.
   - Verdict: covered. A search of the `landing` stencils found no hard-coded skill list.

2. **`conflictSkills` (`internal/mergeresolve/spec.go`).**
   - Set found:
     - `mergeresolve/doc.go:33`, which was updated.
     - `mergeresolve_test.go`, three rows, which were updated.
     - `publish_integration_test.go:94`, which was updated.
     - `finalize_integration_test.go:188`, which was updated.
   - Verdict: covered. No other reader or asserter of the conflict session's skills turned up.

3. **Sampler `skills` (`internal/samplerengine/spec.go`).**
   - Set found:
     - The `skills` comment, which was updated.
     - `spec_test.go:45`, which was updated.
     - No sampler doc or stencil lists the skills: a grep of `contracts/stencils/sampler`, `internal/samplerengine/*.go` and `docs/overview.md` found only `spec.go`.
   - Verdict: covered.

4. **Repository rules, PATTERN and commit message.**
   - **PATTERN entries:** none of them governs skill lists, and `PATTERN-parent-directive` says a role's skills go on the launch spec, which this diff does. The directive declares no PATTERN changes.
   - **Commit message:** it carries no `Pattern-Departure:` trailer, and none is needed.
   - **`AppendSkills`:** the `internal/languageconfig/config.go:143-161` contract says it returns a fresh slice and never writes `own`, which matches the focus entry on aliasing.
   - Verdict: covered.

**Findings:** none.

**Out of lens (noted, not a finding):** `fixReviewTable` in `internal/landingshed/fixreview.go:166` still sets `Skills: in.LanguageSkills`, with no base skills. The summary lists it as a seat that writes no code. That is a scope and judgment call for the handler.

**Tool calls:**
1. One grep of the sampler and landing stencils, `docs/overview.md`, the `samplerengine` and `landingshed` docs and `PATTERN.md`/`pattern/` for skill lists. My context did not hold those files.
