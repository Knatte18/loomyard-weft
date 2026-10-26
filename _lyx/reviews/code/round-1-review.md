---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/discussionparser/doc.go:11
    summary: Package doc still says Validate refuses a bullet whose entry line runs past the line cap, but the check now measures the rule text against the rule-text cap.
    fix: In internal/discussionparser/doc.go line 11, reword the sentence to say Validate refuses an `adds` or `changes` bullet whose rule text, the entry text without a trailing `(test)` or background link, would run past the PATTERN format checker's rule-text cap, so the overrun fails at Discussion and at `lyx loom decision add` rather than first at the plan gate.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/discussionparser/validate.go:61
    summary: Validate's doc comment describes the length check as measuring an overview line against what an entry line may be, but it measures the rule text.
    fix: In the Validate doc comment at internal/discussionparser/validate.go line 61, replace "whose overview line is longer than a PATTERN.md entry line may be" with "whose rule text is longer than a PATTERN.md entry's rule text may be".
---

### [MEDIUM] Stale package doc names the old whole-line cap

**Location:** internal/discussionparser/doc.go:11
**Issue:** The decision record's scope lists the package docs, and CLAUDE.md requires a doc to change in the same commit as the behavior. The diff updates the `pattern` and `planparser` docs and the `patternchanges.go` header, but leaves this sentence describing the entry line and the line cap. `patternchanges.go:152-183` measures the rule text against `patternEntryTextCap`.

### [LOW] Stale Validate comment describes the overview line

**Location:** internal/discussionparser/validate.go:61
**Issue:** The comment says the length check yields a Finding per bullet whose overview line is longer than an entry line may be. The check no longer measures an overview line.

## Rejected

- Cut of a name followed by no ` — ` separator keeps a leading space (code-correctness): at most one character of slack, the format check flags the missing separator separately, and no behavior the answer key asks for depends on it.

## Verified against the focus directive

- `EntryRuleText` and `patternRuleText` cut the same text on rows with and without `(test)`, with and without a link, with both, and with a text-only line. `TestPatternEntryTextCap_EqualsThePatternCap` pins the shared rows. The name-less row exists only in `TestEntryRuleText`, because `patternRuleText` takes bullet text and never sees a name-less line.
- A bullet appended by `lyx loom decision add --pattern-text` reaches `patternOverCapFindings` through the record check. The `PatternTextOverCap` row of `TestAppendDecision_FindingRestoresRecord` covers the refusal.
- No code, spec, stencil or test still names `MaxEntryLineChars`, `patternEntryLineCap` or the 360 whole-line cap. Remaining hits are `docs/research` measurement records and a test comment that names the old cap on purpose. The two stale doc sentences are F1 and F2.
- A prescribed line passes when only its name and link push it past 360, and fails when its rule text exceeds 280, in `TestValidate_PatternEntryLineCap`.
- The new tests add rows to existing tests. The one new function, `TestEntryRuleText`, covers the new exported function and duplicates no existing test.
