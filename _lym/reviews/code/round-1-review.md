---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/heddleengine/shipped.json:5
    summary: The lym-batching override bullet says "the rule above", which names only the preceding read-whole bullet, while the old text tied the override to the never-grep and never-slice rules as well
    fix: In the lym-batching text in shipped.json, replace the bullet "Follow the rule above over the Read tool's own advice to read only the part you need." with one that names its rule, for example "Read a file whole, or every span you need from it, even though the Read tool advises reading only the part you need." Keep the two "Never" bullets after it unchanged.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: internal/heddleengine/shipped.json:13
    summary: Step 4 of lym-plan-edits-before-gate renders its read, decide, edit, rerun sequence as unordered sub-bullets, though the old text and the Goal keep a numbered list where order is the point
    fix: In the lym-plan-edits-before-gate text in shipped.json, number step 4's four sub-bullets ("   1. Read what the fixes need in one message." through "   4. Run the gate or validator again.") instead of using "   - " markers, keeping the 3-space indent under step 4.
---

### [LOW] Override bullet in lym-batching points at "the rule above"

**Location:** internal/heddleengine/shipped.json:5
**Issue:** The old paragraph said "This overrides the Read tool's own advice to read only the part you need: never grep for a line number and then read a slice around it, and never read one file in slices over several turns", so the override covered the whole rule.
The new bullet "Follow the rule above over the Read tool's own advice" resolves by position to the single read-whole bullet before it, and the two "Never" bullets after it sit outside the override.
Every instruction survives and the meaning is still recoverable from the adjacent bullets, so this is a wording weakening, not a dropped rule.

### [LOW] Step 4's sub-steps lose their order markers

**Location:** internal/heddleengine/shipped.json:13
**Issue:** The old step 4 gave "read what the fixes need in one message, decide every fix, make all of them in one message, then run it again", a sequence where order is the point.
The Goal says a numbered list stays numbered where order is the point.
The new step 4 turns that sequence into four unordered sub-bullets under a numbered parent.
The rendered file still reads in order, so the meaning survives, but the order is no longer marked.

## Rejected

- Exact-heading pin in `ShippedEntriesArePinned` is literal prose (lens:code-tests, NIT): the pin protects the structural contract the Goal asks for, a `##` first line, and a reword forcing a test edit is the intended trade.
- No test forbids a `#` or `###` line inside an entry (lens:code-tests): the diff adds none, and the Goal asks only for the heading structure.
- `readSliceNudge` in `batchingnudge.go:101` and `webster-body-implementer.md:54` restate the slice rule in their own words (lens:code-affected-set): neither quotes the old shipped sentences, so the Goal's "quotes the old sentences" clause does not reach them.
- `docs/research/turn-audit-2026-10-28/report.md:228` quotes an older sentence (handler): a measurement report of a past run, left as written.
- `doc.go` sentence about level-two headings as a rule belonging in PATTERN (handler): it describes how the package renders, and a test pins it, so it is not a rule other code must follow.
- Slice rule now stated only in `lym-batching` (handler): both entries target `claude-file` alone and render into the same file, so the rule is still present once and the round entry loses no other instruction by it.
