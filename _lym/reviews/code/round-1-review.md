---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/discussionparser/decisionids.go#continuesItem
    summary: "continuesItem ends a bullet at any unindented line opening with # or <, wider than the heading or HTML block its doc comment and the spec promise, so a continuation opening with an issue number, an autolink or a placeholder is still cut"
    fix: "In continuesItem, exclude a `#` line only when it is an ATX heading (one to six `#` followed by a space or the line's end), and a `<` line only when it opens an HTML block that can interrupt a paragraph (`<!--`, `<?`, `<!` plus a letter, `<![CDATA[`, or `<` or `</` followed by a block-level tag name such as div, p, table, pre, script, style, details). Keep `>` as is. Add TestSectionBullets rows for `- a\\n#123 b\\n- c` wanting [\"a\\n#123 b\", \"c\"] and `- a\\n<slug> b\\n- c` wanting [\"a\\n<slug> b\", \"c\"]."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/discussionparser/partsplice_test.go#TestSectionBullets
    summary: "No test row fails when the HTML-block or blockquote exclusion of continuesItem is removed: CarryOverBlockSkipped has a blank line before its marker, and no row puts a `>` line under a bullet"
    fix: "Add TestSectionBullets rows `## Open risks\\n\\n- a\\n<!-- note -->\\n- b\\n` wanting [\"a\", \"b\"] and `## Open risks\\n\\n- a\\n> quote\\n- b\\n` wanting [\"a\", \"b\"]; name the ending conditions they pin in the test's doc comment."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:45
    summary: "The Merge danger blast radius names only discussionparser, though the change alters the bullet text the raddle-propose gate (raddleparser coverage), SplicePart part records and the acceptance coverage check in every discussion gate and validate-discussion compare; a proposal or part record carrying the cut text, or a binary on the old rule reading a record, now disagrees"
    fix: "In _lym/landing/summary.md's Merge danger section, widen the blast radius to name raddleparser's criterion and risk coverage, SplicePart's part records and CheckDecisionIDs' acceptance coverage, and state that an in-flight raddle proposal holding a cut criterion now fails criterion-unknown and criterion-uncarried; reconsider the two-way call against a binary on the old rule reading the same record."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/discussionparser/decisionids.go#continuesItem
    summary: "An unindented thematic break (---, ***, ___) right after a bullet joins its text, though markdown ends the list there, contradicting the doc comment's 'as markdown reads a lazy continuation'"
    fix: "In continuesItem, return false for a line of three or more of the same `-`, `*` or `_` with only spaces between; add a TestSectionBullets row `## Open risks\\n\\n- a\\n---\\n- b\\n` wanting [\"a\", \"b\"]."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/discussionparser/partsplice_test.go:58
    summary: The summary's claim that a spliced part record reads back the same text is untested for the reported case, since the splice fixture's only continuation line is indented
    fix: "In spliceRecord's `## Acceptance criteria`, make one criterion run onto an unindented second line (for example `- D1: first holds\\nfor every call.`) and carry its newline-joined text in TestSplicePart's part.Criteria, so the existing read-back assertion pins the round trip."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-discussion-spec.md:62
    summary: "The two new sentences sit between the whole-word rule and the 'So D1 is not met by D10' sentence that follows from it, breaking that reasoning"
    fix: "Move the two new sentences in contracts/specs/loom-discussion-spec.md to after the sentence opening 'So D1 is not met by D10', keeping their text as F1's fix leaves the code."
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/discussionparser/partsplice_test.go:79
    summary: TestSectionBullets' doc comment does not name the unindented continuation or the heading that ends a bullet, which its new rows pin
    fix: "Extend the doc comment of TestSectionBullets to name an unindented continuation line and each line that ends a bullet, matching its rows after F2 and F4."
---

### [MEDIUM] continuesItem cuts more than markdown does

**Location:** `internal/discussionparser/decisionids.go:203`

**Issue:** `continuesItem` rejects every unindented line whose first byte is `#` or `<`.
Markdown ends a paragraph only at an ATX heading (`#` to `######` then a space or the line's end) and at an HTML block of types 1 to 6; a type-7 tag, an autolink or a placeholder such as `<slug>` cannot interrupt a paragraph.
So `- a\n#123 b` and `- a\n<slug> b` are still cut at their first line, the defect the board entry reports, and `bulletLines` counts an id named only there as uncovered.
The doc comment (`decisionids.go:196`), `SectionBullets`' doc (`partsplice.go:38`) and the new spec sentence all claim the narrower rule.

### [MEDIUM] HTML-block and blockquote exclusions are unpinned

**Location:** `internal/discussionparser/partsplice_test.go:93`

**Issue:** `CarryOverBlockSkipped` has a blank line between the bullet and the marker, and the carry-over skip flushes the bullet on its own, so it passes with `<` dropped from the exclusion.
No row of `TestSectionBullets` or `TestCheckDecisionIDs` puts a `>` line under a bullet.
The summary justifies the `<` exclusion by keeping a carry-over marker or comment out of the bullet above it; nothing pins that.

### [MEDIUM] Merge danger blast radius names less than the change can break

**Location:** `_lym/landing/summary.md:45`

**Issue:** The blast radius reads `discussionparser`.
The changed bullet text is what `raddleparser`'s `coverage` (`internal/raddleparser/check.go:179`) matches exactly, what `SplicePart`'s `requireBullets` checks, and what `CheckDecisionIDs` reads in the discussion gates, Darn-Write's guard and `lym loom validate-discussion`.
A stored record with unindented prose directly under a bullet now yields a longer bullet, so a raddle proposal filed on the old reading fails `criterion-unknown` and `criterion-uncarried`, and a binary on the old rule refuses a proposal filed on the new one.

### [LOW] A thematic break joins the bullet

**Location:** `internal/discussionparser/decisionids.go:203`

**Issue:** `continuesItem("---")` is true, so `- a\n---` reads as `a\n---`.
Markdown ends the list at a thematic break, as the doc comment's lazy-continuation claim implies.

### [LOW] Splice round trip of an unindented continuation is untested

**Location:** `internal/discussionparser/partsplice_test.go:58`

**Issue:** `summary.md:24` claims a spliced part record reads back the same text.
The fixture's only continuation, `Risk one\n  continues here.`, is indented, so `requireBullets` and the `renderBullets` read-back never run on the reported shape.

### [LOW] Spec sentences break the whole-word reasoning

**Location:** `contracts/specs/loom-discussion-spec.md:62`

**Issue:** The inserted sentences now precede "So `D1` is not met by `D10`…", which follows from the whole-word rule above them, not from where a bullet ends.

### [NIT] TestSectionBullets doc comment lags its rows

**Location:** `internal/discussionparser/partsplice_test.go:79`

**Issue:** The doc comment lists plain and continuation bullets, a fenced bullet, a carry-over block and an absent heading; it does not name the unindented continuation or the heading that ends a bullet.

## Rejected

- code-scope reported no finding; every hunk traces to the board entry's parser option, and none was rejected.
- The two MEDIUM findings of code-tests, one for `<` and one for `>`, are one defect with one fix site and are kept merged as F2.

## Focus departures

None. Each focus entry was reached:
both readers end a bullet at the same places for fences, carry-over blocks, blank and whitespace-only lines, indented markers and section boundaries;
after the diff the raddle-propose gate accepts the newline-joined text `D6: …\nonly inside the area.`, and no other reader of decision record bullets keeps the indented-only rule (`internal/partsparser` reads a different file).
