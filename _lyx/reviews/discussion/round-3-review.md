---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:67
    summary: D3 and Scope say loom-template-plan.md carries the granularity sentence, but that stencil carries no granularity rule, only a pointer to it at line 21
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:65-70
    summary: D3 gives no replacement for loom-plan-spec.md's private-supporting-type sentence, though the acceptance list removes its phrase "independent meaning or testability"
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:196
    summary: The removed-phrase lead sentence uses forms the files do not carry, such as bare "independently testable" and "independently reviewable/testable unit" for the spec, which spells it "reviewable and testable"
---

### [MEDIUM] D3 names a granularity sentence in `loom-template-plan.md` that does not exist

**Location:** `_lyx/discussion/decision-record.md:67` (D3), and Scope's `loom-template-plan.md` bullet at line 15 ("the same two rules").
**Issue:** D3 orders "independently testable or reusable gets its own card" replaced in `loom-plan-spec.md`, `loom-template-plan.md` and the Plan-Review rubric.
The phrase exists in the spec (line 301) and, as "an independently testable symbol gets its own card", in the rubric (line 46).
`loom-template-plan.md` holds no granularity rule: grep for granular, reusable and independent finds only line 21, which says `scribe:testing` is loaded because "the card-granularity rule and the bundle-your-own-test rule" are testing judgments, and line 70's "independently committable".
The acceptance list agrees with the file: it lists no granularity phrase for the stencil.
A plan writer following D3 would hunt for text that is not there, or invent a granularity rule in the stencil.
**Fix:** Drop `loom-template-plan.md` from D3 and say the stencil carries no granularity rule.
Narrow Scope's stencil bullet to the criterion rule, the "normal shape" sentence and the Step 0 wording.

### [LOW] D3 leaves the private-supporting-type sentence unreplaced

**Location:** `_lyx/discussion/decision-record.md:65-70` against `contracts/specs/loom-plan-spec.md:300`.
**Issue:** Spec line 300 reads "A symbol with no independent meaning or testability apart from another symbol in the same card ... is bundled into that other symbol's card."
The acceptance list requires the phrase "independent meaning or testability" gone, but D3 states nothing for this sentence.
**Fix:** Add to D3 that this sentence drops "or testability": a symbol with no independent meaning apart from another in the same card is bundled into that card.

### [LOW] Removed-phrase lead sentence does not match the quoted phrases

**Location:** `_lyx/discussion/decision-record.md:196`.
**Issue:** The lead sentence names "independently testable" (symbol or reusable) and "independently reviewable/testable unit" for all three files.
The spec spells the latter "independently reviewable and testable unit"; only the rubric spells it with a slash.
The per-file list below holds the exact quotes, so the lead sentence contradicts it and invites a wrong grep.
**Fix:** Replace the lead sentence's phrase list with a reference to the per-file phrases below it.

## Focus departures

None.
Settled verdicts on the three focus questions:

- D11: the Decision itself states the bound (a file-level comment excuses only tests touching the state it names; a test touching none calls `t.Parallel`). Not a defect.
- Removed-phrase list: every quoted phrase exists verbatim in the file named for it (checked with `grep -F`); the gaps are F1 and F3, and D3's replacement for the spec's line 300 is F2.
- D1, D10, D11, D12, D13 versus Scope: each Decision opens "Landed in the Discussion commit (D13)", Scope lists them as already done, and D13 is in the past tense. Not a defect.
