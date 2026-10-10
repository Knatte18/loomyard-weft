# Card 11 — pattern-entry-line-cap

**Create:**
- `plan:internal/planparser#checkPatternEntryLineCap` -> `func checkPatternEntryLineCap(plan *Plan) []ValidationError`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser/validate_test.go`
- `internal/planparser/doc.go`
- `//contracts/specs/loom-plan-spec.md`
- `//contracts/stencils/loom/loom-template-plan.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/pattern#MaxEntryLineChars`
- `internal/planparser#Card`

**Intent:** Fail the plan gate on a card that prescribes a verbatim `PATTERN.md` entry line over the cap (D13).
`checkPatternEntryLineCap` reads each card's file bytes through its `SourcePath`, scans only fenced code blocks, and flags every line inside one that starts with `` - `PATTERN- `` and exceeds `pattern.MaxEntryLineChars` runes, as a blocking `pattern-entry-line-cap` finding naming the entry, the length and the cap, with the way forward to shorten the line or move the detail to the entry's background file.
An entry line carries backticks of its own, so no inline span can hold one whole, and inline spans are never scanned.
The check reads raw card text and never a second parse of the plan (PATTERN-sole-parsers).
`validate` appends it after `checkVerifyModuleWide`, so every entry point but `plan-unapproved`'s reports it; `doc.go`'s check list and the spec's validation-checks table gain the row.
`loom-template-plan.md` tells the plan writer that a prescribed addition to an entry line must leave it under the cap, in the card rules beside the attack-surface sentence.
Bound: the check measures verbatim lines only; a prose instruction to append a sentence is the stencil's rule.
A new `TestValidate_PatternEntryLineCap` pins a fenced entry line over the cap failing and one under the cap passing, plus an over-cap line outside a fence passing; the planparser row of `test-budget.yaml` rises by one.

**ImpactSummary:** One new check in planparser's dispatch list; no existing check or signature changes.
