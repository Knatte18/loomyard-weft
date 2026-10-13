---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/landing/summary.md:22
    summary: Summary says a batch with no entries "got a non-nil empty pack" before, but such a batch got a pack with one empty subsection per card and the pack-on prompt, so the empty-pack change alters its prompt
    fix: In the "Design decisions" bullet of _lyx/landing/summary.md, replace "Before, such a batch got a non-nil empty pack." with a statement that such a batch previously rendered the pack-on prompt, with the one-turn read instruction and a "none" or "names no member glyph" subsection per card, and now renders the pack-off prompt; also qualify "Without a pack the prompt is unchanged" so it does not cover that batch.
---

### [LOW] Summary misdescribes the prior behavior of an all-empty pack

**Location:** `_lyx/landing/summary.md:22`, against `internal/websterengine/beginbatch.go:419`

**Issue:** The old code turned only a `nil` index result into `[]planindex.CardPack{}`.
An index that returned one `CardPack` per card with no resolved or unresolved entry, as the new `beginbatch_test.go` row builds, stayed a non-nil pack.
It rendered the pack-on prompt: the pack locate fragment and the pack section with its `none` or "names no member glyph" subsections.
The new `slices.ContainsFunc` guard returns nil for it, so that batch's prompt changes, and the summary's "Without a pack the prompt is unchanged" does not hold for it.
The board entry lists "empty pack" among the cases that read as today, so the code change is licensed.
Only the summary's account of the prior behavior is inaccurate.

## Judged and found sound

- **Empty-pack scope:** the board entry names "empty pack" beside pack off and language none as the no-pack cases, so the nil return and the `kickstart_pack=empty` log value fall within it.
  The prompt did not already render as pack-off for an all-empty batch (see F1).
- **Pack last:** `{{.kickstart_pack}}` now ends `webster-body-implementer.md` after the batch-report section, and the pack section opens with the one-turn read instruction.
  `claims_test.go` pins the marker position, `template_test.go` pins the report-then-pack order, and `beginbatch_test.go` pins that the pack heading is the last heading of the written prompt file.
  These cover different levels (stencil, renderer, begin-batch file) and are not redundant.
- **Batch-report wording:** the section is unchanged and still says the report is the fork's last action, as the entry asks; only its place in the text moved.
- **No-pack prompts:** only `webster-fragment-fresh-read-pack.md` and the pack section changed among the fragments, so the plain FRESH-READ text, which carries no pointer to a pack section, is byte-identical.
  The pointer to "the last section of this prompt" appears only in the pack-on fragment.
- **Docs:** `doc.go` and the `WebsterFragmentPackSection` and `RenderForkPrompt` comments match the new placement and the four log values.
