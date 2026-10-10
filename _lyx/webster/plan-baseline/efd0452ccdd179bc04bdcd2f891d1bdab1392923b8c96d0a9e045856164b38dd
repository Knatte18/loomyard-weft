# Card 11 — pattern-entry-line-cap

**Create:**
- `internal/planparser#checkPatternEntryLineCap`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser#Card`
- `internal/planparser/validate_test.go`
- `internal/planparser/doc.go`
- `internal/loomcli/parity_test.go`
- `//contracts/specs/loom-plan-spec.md`
- `//contracts/stencils/loom/loom-template-plan.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/pattern#MaxEntryLineChars`

**Intent:** Fail the plan gate on a card that prescribes a verbatim `PATTERN.md` entry line over the cap (D13).
`checkPatternEntryLineCap` scans each card's `Text`, the raw text of its file the parser already holds, so the check reads no file and adds no I/O path; it scans only fenced code blocks and flags every line inside one that starts with `` - `PATTERN- `` and exceeds `pattern.MaxEntryLineChars` runes, as a blocking `pattern-entry-line-cap` finding naming the entry, the length and the cap, with the way forward to shorten the line or move the detail to the entry's background file.
`Card`'s `Text` field comment names this check beside `CheckCardFabricReference` as its readers.
An entry line carries backticks of its own, so no inline span can hold one whole, and inline spans are never scanned.
The check reads raw card text and never a second parse of the plan (PATTERN-sole-parsers).
`planparser` gains its import of `internal/pattern` for the cap, which brings `pattern`'s own dependencies (`stencil`, `stencilstore` and through them `lyxcwd`, `gitexec`, `proc` and `logger`) into its build graph without a call into any of them; no cycle results, since none of them imports `planparser`.
`doc.go`'s sentences that the package never imports `lyxcwd` and stays a tier1-pure leaf are reworded to what still holds: `planparser` itself never resolves a cwd or spawns a process, and its loomyard imports are `lyxdirs` and `pattern`, the latter for the entry-line cap alone.
`validate` appends it after `checkVerifyModuleWide`, so every entry point but `plan-unapproved`'s reports it; `doc.go`'s check list and the spec's validation-checks table gain the row.
`loom-template-plan.md` tells the plan writer that a prescribed addition to an entry line must leave it under the cap, in the card rules beside the attack-surface sentence.
Bound: the check measures verbatim lines only; a prose instruction to append a sentence is the stencil's rule.
A new `TestValidate_PatternEntryLineCap` pins a fenced entry line one rune over the cap failing with the entry, length and cap named, one at exactly the cap passing, a line under the rune cap whose multibyte runes put its byte length over the cap passing, an over-cap line outside a fence passing, an over-cap line in a fence whose opening line carries an info string such as `markdown` failing, an over-cap line after an unclosed fence failing, and an over-cap line in the second of two fenced blocks of one card failing while the prose between them is not scanned; the planparser row of `test-budget.yaml` rises by one.
`TestGateParity_PlanGate` in `parity_test.go` gains a `PatternEntryLineCap` fixture whose card fences an over-cap entry line, on which the gate and `validate-plan` both report `pattern-entry-line-cap`.

**ImpactSummary:** One new check in planparser's dispatch list and one field comment; no existing check or signature changes.
