---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/websterengine/render.go:313
    summary: The diff moves the one-turn read instruction and the pack into a separate pack file and ends the prompt with a pointer line, where the board entry asks for them as the prompt's own last section.
    fix: Drop the pack file and render the one-turn instruction followed by the pack inline, as the prompt's last section, through the body's `kickstart_pack` marker. In `render.go`, make `renderPackRead` produce that section from `webster-fragment-pack-section`, so the instruction heads it and the card subsections follow. Remove the `packPath` parameter and the `packFile` return from `RenderForkPrompt`, and remove the `packPath` resolution and pack-file write from `BeginBatch` in `beginbatch.go`. Delete `webster-fragment-pack-read.md` with its `stencils.go`, `.gitattributes`, `markers_test.go` and `claims_test.go` entries. Keep the batch-report section above the new last section, keep the pack-on FRESH-READ rule pointing at the final section instead of carrying the instruction, and adjust `doc.go`, `beginbatch_test.go` and `template_test.go` to the inline layout.
---

### [BLOCKING] Pack and read instruction live in a separate file, not the prompt's last section

**Location:** internal/websterengine/render.go:313, internal/websterengine/beginbatch.go:558-590, contracts/stencils/webster/webster-fragment-pack-read.md
**Issue:** The board entry's Change section says that with a pack the prompt's last section is the one-turn instruction ("read all of this in one turn, in parallel, before any edit") followed by the pack.
The diff does not do that.
It writes the instruction and the pack to `<batch>-pack.md`, and the prompt's last section is a `## Start here` heading with the line "Now read `<pack file>` and do what it says."
The entry licenses neither the new file, the new `RenderForkPrompt` signature (`packPath` in, `packFile` out), the new `webster-fragment-pack-read` stencil, nor the new begin-batch write step.
The detour also works against the entry's goal.
The fork's first action is now a Read of the pack file, and only after that does "in one turn, in parallel" apply, so the parallel reads move to the second tool turn.
The entry's measure, the share of forks with three or more parallel reads in the first work turn, cannot show the intended gain when the pack file's content arrives only in that first turn's result.
The entry's other points are met: the batch-report section stays above the read section and still names the report as the last action, a render without a pack ends as before, and the recovery prompt takes no pack.
The empty-pack handling (`kickstart_pack=empty`, no pack) is licensed by the entry's "empty pack" case.
