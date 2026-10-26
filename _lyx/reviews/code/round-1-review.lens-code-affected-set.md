```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set I found and whether it is covered:

- **`checklistBoxHalted` moves from `[-]` to `[=]`.**
  - Set: `checklist_test.go` rows that pin a halted current producer, and the doc and stencil text that names the old glyph.
  - Covered. The diff updates all five test rows that asserted `[-]` for a halted producer.
  - Grep for `[-]` and `[=]` over the repo, excluding `_lyx`, `.lyx` and `.git`, finds only `checklist.go`, `checklist_test.go` and `doc.go`.
  - No other view, stencil, doc or fixture reads the halted glyph.
  - `doc.go` never named the old halted glyph, so the move leaves it with nothing stale.
- **`checklistBox` gains a `skipped bool` parameter.**
  - Set: callers of `checklistBox`.
  - Covered. There are two production callers, both updated: the plain-row loop at `checklist.go:272` and the visited-segment heading at `checklist.go:428`, which passes `false` on purpose.
  - The grep found no other caller outside the tests.
- **`checklistReviewBlock` gains a `skipped` parameter.**
  - Set: callers of `checklistReviewBlock`.
  - Covered. The one caller, at `checklist.go:268`, passes `skipped`.
- **New `checklistSkippedRows` and the skipped box on an unvisited segment heading.**
  - Set: every renderer path that draws a box.
  - Covered. The other path in `renderChecklist` (the lines up to 231) is a separate early-return view. It calls neither `checklistBox` nor the review block, as far as the context I hold shows, so it needs no change.
- **Documentation.**
  - Set: `doc.go`'s checklist paragraph.
  - Covered. `doc.go:21` now names `[-]`.
  - It does not name the halted `[=]` box, but it never named a halted box before either. The task's scope only asks for the skipped box, so I raise no finding.
  - `PATTERN.md`, `pattern/`, `docs/` and `contracts/` hold no reference to the checklist boxes.
- **Stencils.** No stencil references the checklist boxes. Stencil wording does not apply to this diff, since it changes none.
- **Fixtures.** No fixture or golden file asserts the box glyphs. The only asserting file outside `checklist.go` is `checklist_test.go`.

No findings.

Tool calls (2):
1. Grep for `[-]` and `[=]` repo-wide, excluding `_lyx`, `.lyx` and `.git`. My context only had the earlier grep restricted to `.go`, `.md` and `.yaml`, so this one covers other file types and stencils.
2. Grep for checklist, box, halted and "not reached" wording under `docs`, `PATTERN.md`, `pattern`, `contracts` and `internal`. I needed it to find prose or docs that describe the boxes.
