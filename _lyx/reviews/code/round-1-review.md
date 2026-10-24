---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: contracts/specs/loom-discussion-spec.md:38
    summary: The spec says "The format check refuses that line" without naming the discussion check or the pattern-change-over-cap finding, and "format check" also reads as the PATTERN format checker
    fix: In contracts/specs/loom-discussion-spec.md line 38, replace "The format check refuses that line when it is longer" with "The discussion check refuses that line, as a pattern-change-over-cap finding, when it is longer", keeping the rest of the sentence.
    origin: lens:code-affected-set
---

The diff makes the change the amended board entry asks for.
`Validate` reports one `pattern-change-over-cap` finding per `adds` or `changes` bullet whose overview line runs past 360 runes, and `AppendDecision` restores the record on that finding through the existing restore-on-finding path.
The measured line `- \`<ID>\` — <text>` matches what `pattern-entry-line-cap` measures, with the text trimmed and the appended-after-Discussion suffix excluded.
`ParsePatternChanges`, the plan gate, the PATTERN format checker and the cap value are untouched.
Tests cover the at-cap pass, one rune over, multi-byte runes, `removes` never measured, the appended-bullet way forward and the cap pin, without duplication.
The commit carries no `Pattern-Departure:` trailer and needs none, and the diff declares no PATTERN change.

### [NIT] Spec sentence does not name the check

**Location:** contracts/specs/loom-discussion-spec.md:38
**Issue:** The new sentence calls the refusing party "the format check", which reads as the PATTERN format checker and never names the discussion check or its `pattern-change-over-cap` finding.

## Rejected

- code-tests NIT on the `textFor` helper omitting the `adds: ` prefix: the rows are correct, the fork itself called it not a defect.
- code-pattern-fit note on the `Co-Authored-By` trailer naming a different model than the session attribution: outside the rubric's findings.
- code-correctness note on a whitespace-only suffix author giving an empty `By`: the suffix regex requires a non-empty author and `AppendDecision` supplies one, so the case is unreachable.
