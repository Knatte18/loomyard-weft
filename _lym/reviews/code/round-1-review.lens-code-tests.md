```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails when it breaks.

**Behaviour list**

1. **The commit sentence in `fixScopeRules` (source scope) now reads "commit each finding's fix on its own".**
   - Pinned by: none.
   - The `source` subtest of `TestComposePrompt_FixScope` (`internal/burlerengine/prompt_test.go:335-344`) checks only for `"commit"` and the absence of `"no git"`.
   - This is static prompt prose, so it is exempt from the coverage requirement.
2. **The joint-commit group rule, the format `fix <finding-id>, <finding-id> — …`, and the rule that a finding sharing no changed file keeps its own commit.**
   - Pinned by: none.
   - These are fixed strings returned on the source branch and passed into the stencil, with no input-dependent logic.
   - They count as static prose, so they are exempt.
   - An assertion on them would itself be a test-economy finding.
3. **The branch selection of `fixScopeRules` (source vs overlay).**
   - Pinned by the `source` and `overlay` subtests of `TestComposePrompt_FixScope`, through `"no git"` present or absent.
   - The diff leaves this behaviour unchanged.
4. **The `doc.go` FixScopeSource bullet.** Doc prose, exempt.
5. **The `site not found —` marker in `burler-review-format.md` and `burler-step-fix.md`.**
   - Static stencil prose, exempt.
   - `verdict.go` does not parse it, so there is no code behaviour to pin.
6. **The mutation report** shows 0 mutants on changed lines, consistent with the diff changing only string literals.

**Findings**

- **F1**
  - severity: LOW
  - class: consistency
  - location: `internal/burlerengine/prompt_test.go#TestComposePrompt_FixScope`
  - summary: The `overlay` subtest's `requireNotContains(t, got, "commit each fix")` now checks nothing. The diff removed the phrase "commit each fix" from the source-scope prose, and no stencil holds it, so the assertion cannot fail whatever the overlay output says. The test's doc comment still says source output "carries the commit-per-fix phrasing", but the source subtest asserts only `"commit"`.
  - evidence: `internal/burlerengine/prompt_test.go:330`, `internal/burlerengine/prompt_test.go:355`, `internal/burlerengine/prompt.go:328`. The earlier grep found "commit each fix" only in `prompt_test.go:355` and in an unrelated webster stencil.
  - fix: Drop the dead `requireNotContains(t, got, "commit each fix")` line, since the `"no git"` checks already pin the branch split. Reword the doc comment so it no longer claims a "commit-per-fix phrasing" assertion.

I made no tool calls. All evidence came from the context I inherited: the test span from `sed` of `prompt_test.go:320-370`, the `prompt.go` span, and the repo-wide grep for "commit each fix".
