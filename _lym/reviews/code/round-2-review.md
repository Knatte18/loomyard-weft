---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/planparser/renumber.go:625
    summary: cardMentionRe opens with a word boundary, which a hyphen satisfies, so a command example such as insert-card 3 or remove-card 3 in plan prose is read as a bare card reference and rewritten
    fix: >-
      In rewriteCardMentions (internal/planparser/renumber.go), skip a cardMentionRe span whose preceding byte in masked is a hyphen, that is span[0] > 0 && masked[span[0]-1] == '-', the same way a span is already skipped when mapsFaithfully is false, but without marking the line kept. Add one row to TestRenumber in internal/planparser/renumber_test.go (or one line in the TestRenumber_RewritesCardIDs fixture) holding `lym loom insert-card 3 slug` in a card's Intent and asserting the line is unchanged and unlisted after an insert at position 2.
  - id: F2
    severity: LOW
    class: consistency
    location: contracts/specs/loom-plan-spec.md:227
    summary: the spec says a card id is never reported, yet a rewritten card id is reported under rewritten_refs
    fix: >-
      In contracts/specs/loom-plan-spec.md line 227, replace the sentence beginning "A card id is never reported:" with "A card id never needs a hand fix: both verbs rewrite one naming a moved card and leave one naming the removed card, so a card refers to another card by its card id."
---

### [LOW] Hyphenated command words read as a bare card reference

**Location:** internal/planparser/renumber.go:625
**Issue:** `cardMentionRe` starts with `\b[Cc]ards?`. Go's `\b` is true between `-` and `c`, so the text `insert-card 3` or `remove-card 3` matches as `card 3`. A plan card that documents or quotes these verbs (this very task's plan would) has the position argument rewritten to the moved number. The base commit had the same regex, but it only reported such a line; the diff now rewrites it, which turns a harmless false report into a silent change of a command example. The case is narrow and a planner sees the change under `rewritten_refs`, so the severity is LOW.

### [LOW] Spec says a card id is never reported, but rewritten card ids are reported

**Location:** contracts/specs/loom-plan-spec.md:227
**Issue:** The sentence "A card id is never reported: both verbs rewrite one naming a moved card and leave one naming the removed card" contradicts the verbs' own output. A rewritten card id is reported under `rewritten_refs` (`RewrittenRef` in internal/planparser/renumber.go and `internal/loomcli/doc.go`), and only a line left holding a card id is not listed under `prose_mentions`. The previous sentence already uses "report" for the `prose_mentions` case, so the reader cannot tell which report is meant. Reword it to say a card id is never left for the planner to fix, for example "A card id never needs a hand fix: both verbs rewrite one naming a moved card and leave one naming the removed card, so a card refers to another card by its card id." This is a spec-wording fix, recorded here as the finding `fix` needs the exact text: replace the sentence at `contracts/specs/loom-plan-spec.md:227` with that wording.

## Rejected

- Focus 1, kept-range clause on insert: reached. `insertSecondMentions` has `Spans: cards 1–3` strictly inside the insert position and `Pair: cards 1–2` ending at it; `insertThirdMentions` has `Covers cards 2–3` and `Spans: cards 1–3` ending at position 3. The boundary row is `Pair: cards 1–2` under an insert at position 3: it is absent from `insertThirdProse` and `insertThirdMentions`, so it is asserted unchanged and unlisted. Not a defect.
- Focus 2, failure reversal for a card changed only by a bare reference: the row "insert: Card Index rewrite after a card changed only by a bare reference" rewrites `01-alpha.md` to hold only `card 4`, lets the card's own write succeed and faults the overview write after it. `requireSameTree` over `fx.root` (which holds the journal directory) then proves alpha is journaled and restored byte for byte and no journal is left. It does not fault alpha's own write, but `TestRenumber_CardIDRewriteFailureReversed` faults a stayed card's write on the same code path, so the pair covers both points. Not a defect.
- Focus 3, Card Index and heading lines: delta's Card Index line holds `after card 3`, a reference that moves; the `wantSummaries` check requires the text unchanged after the insert, and the moved-card `want` check requires the heading to carry only the new number. Both would fail if `cardIndexLines` or `headingLine` stopped skipping. Not a defect.
- Focus 4, `glyphlessFile` rows: a missing file (`format over a test file the repository lacks`) and a syntax error (`format over a test file with a syntax error`) are covered, each keeping the finding. The same-base other-directory row was dropped in ad49827a. quarry's `TOC` on a missing file target returns `ErrTargetNotFound` (quarry v0.4.0, `internal/engine/toc.go`), so `glyphlessFile` returns false before the base-name comparison is reached, and a file with symbols is refused whatever the lookup returns; the dropped row could not discriminate. The `path.Base` comparison is a redundant guard, not a defect.
- Focus 5, spec sentence near line 226: the leave-as-is clause now opens with "A bare reference naming the removed card, or a range ...", so its antecedent is unambiguous, and the card-id rewrite is a separate sentence. Its last sentence is the second finding above.
- Declared PATTERN changes: none declared, and the diff touches neither `PATTERN.md` nor `pattern/`.
- Scope against the decision record: both items of the amended entry are made (a no-symbol test file admitted by `edit-test-file`, bare `card N` references rewritten and the unrewritable ones reported). The exemption lives in the plan index and `planparser` still reads no file, in line with the summary's claim.
- Test economy: no new test duplicates an existing one; the two `TestCheckEditTestFile` rows pin the exempt and the not-exempt path, and the `TestIndex_AppendsPlanChecks` rows pin the three quarry outcomes through the real `glyphlessFile`.
