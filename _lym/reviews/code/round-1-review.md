---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/lenses/config_test.go:TestTemplateChecks_FlagBrokenTemplates
    summary: the writerFanGaps subtest is nested inside the emphasisLineGaps subtest closure, so it runs as emphasisLineGaps/writerFanGaps
    fix: In TestTemplateChecks_FlagBrokenTemplates, move the t.Run("writerFanGaps", ...) block out of the t.Run("emphasisLineGaps", ...) closure so both are direct subtests of the parent test; change no assertion.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/lenses/config_test.go:TestConfigTemplate_DecodesAndIsSelfConsistent
    summary: the board-scope half of the procedureLenses assertion is a tautology, since the test sets procedureLenses["board-scope"] two statements earlier
    fix: In the assertion that reads !procedureLenses["board-scope"] || procedureLenses["leanness"], drop the board-scope half and keep the check that leanness is not a procedure lens, adjusting the error message to match.
---

### [LOW] writerFanGaps subtest is misnested

**Location:** internal/lenses/config_test.go, `TestTemplateChecks_FlagBrokenTemplates`
**Issue:** The `writerFanGaps` `t.Run` sits inside the closure of the `emphasisLineGaps` subtest, so its name reads `emphasisLineGaps/writerFanGaps`. Both are exercised; only the structure and names are wrong.

### [LOW] Tautological board-scope assertion

**Location:** internal/lenses/config_test.go, `TestConfigTemplate_DecodesAndIsSelfConsistent`
**Issue:** `procedureLenses["board-scope"] = true` is set just before `!procedureLenses["board-scope"]` is tested, so that half can never fail. Only the `leanness` half checks anything.

## Rejected

- code-tests F2 (the "no exemption" `writerFanGaps` row duplicates the "lens the writer fan lacks" row): the row is the control for the "with its exemption" row, and it differs in the exemption map, so it is no redundancy.
- code-tests F3 (assertions on static lens prose): the sentences are pinned by D1, D3 and D4's acceptance criteria, and the existing `forced-edits` and `row-validity` pins follow the same pattern; lens prose in `template.yaml` is no stencil.
- code-tests F4 (full template comment in the `webster_*` rows' `wantDocs`): the existing `code_reviewer_model` row in the same test already asserts the comment text, so the new rows follow the file's convention.
- code-tests note that no test pins the csharp `review-code` order: the lens-set loop requires a layer fan to start with the base fan, so `code-pattern-fit` left in place fails, and the order is pinned indirectly; the fork raised it as a note, not a finding.

Holistic review (handler): the diff makes every change D1 to D7 asks for, and the repo-wide grep finds no stale reference to the retired lenses outside `docs/research` measurement reports. The open-key comments in `config.go`, `docs/overview.md` and `pattern/PATTERN-config-layers.md` agree with `ConfigSpec`. No scope creep: the fixer fans, the `twill-plan` fan and the hub's live files are untouched. No `Pattern-Departure:` trailer is needed, and the commits carry none.
