```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
- **Edit-directive ban in both engine copies** (`contracts/stencils/engine/{claude,codex}/edit-directive.md:18-20`): no behaviour test. The diff only changes static stencil prose, which is exempt. `TestDirective_ReadsTheEnginesCopyWithoutBanner` already pins how `Directive` reads and renders each engine's copy. The diff changes no code path.
- **Orch-role ban note** (`contracts/orchrole/orch-role.md:452-453`): none. Static prose, exempt.
- **Package doc** (`internal/editdirective/doc.go:16-19`): none. Comment text, not behaviour.
- **`Directive` rendering behaviour**: the diff leaves it unchanged. The existing rows in `TestDirective_ReadsTheEnginesCopyWithoutBanner` still pin it: engine tool wording, no marker, no leading comment, no trailing newline.

**Findings:**

1. **MEDIUM, consistency.** `internal/editdirective/editdirective_test.go:20-21, 41-43`: the new `scratchBuildBan` const and its `strings.Contains` check assert the static prose of a stencil. The rubric's test-economy bullet names that as a finding: "a new assertion on the static prose of a stencil, spec, doc or other text."
   - Evidence: `stencilkit.Seed` (`internal/testkit/stencilkit/stencilkit.go:26-38`) seeds the shipped registry stencils. The assertion therefore checks the wording of the shipped `edit-directive.md` copies, not any behaviour of `Directive`.
   - It also ties a wording edit in either copy to a test edit.
   - The rubric says the prose check belongs to review: "a test asserting the prose never stands in for this check."
   - Fix: delete the const and the assertion, and revert the test's doc-comment line 16 to its prior wording ("its own tool wording is present, ...").

Tool calls:
- `grep` of `func Seed` in `internal/testkit/stencilkit`: to confirm whether the test reads the shipped stencil prose or a fixture. The manifest did not include `stencilkit`.
