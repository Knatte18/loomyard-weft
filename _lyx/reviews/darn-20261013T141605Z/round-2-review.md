---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/landing/summary.md:1
    summary: The change description claims a separate pack file, a Start here pointer line, a webster-fragment-pack-read stencil and a pack path parameter, none of which the diff contains
    fix: Rewrite _lyx/landing/summary.md to describe the diff as committed. The pack section is filled inline into the kickstart_pack marker at the very end of webster-body-implementer.md, after the batch-report section, and opens with the line "Read all of this in one turn, in parallel, before any edit". No pack file is written, RenderForkPrompt takes no pack path and returns only the prompt, there is no Start here heading and no webster-fragment-pack-read stencil. Keep the claims that do hold true. These are that FRESH-READ points at the pack section, that an empty pack renders as the pack-off case and logs kickstart_pack=empty, that the prompt is byte-identical without a pack, and that the recovery prompt never takes a pack. Drop the title wording "in its own file, read line last" and "by reference".
  - id: F2
    severity: LOW
    class: scope
    location: internal/websterengine/beginbatch.go:561
    summary: The promptPath computation was moved above buildKickstartPack and RenderForkPrompt, which no longer need it, so the move is an unlicensed leftover of the abandoned pack-file design
    fix: In BeginBatch, move the block `promptPath, err := filepath.Abs(filepath.Join(deps.Geom.PromptsDir, batchName+".md"))` with its error return back to where it was, after the `os.MkdirAll(deps.Geom.PromptsDir, 0o755)` block and before the `os.WriteFile(promptPath, ...)` call, so beginbatch.go has no diff hunk there.
---

### [MEDIUM] Change description describes a design the diff does not contain

**Location:** `_lyx/landing/summary.md:1`
**Issue:** The summary says begin-batch writes `<batch>-pack.md`, that the prompt's last line under `## Start here` is "Now read `<pack file>` and do what it says", that a new stencil `webster-fragment-pack-read` fills the marker, and that `RenderForkPrompt` takes the pack path and returns the pack content.
The committed code does none of this.
`render.go` has no pack path parameter and returns only the prompt, `beginbatch.go` writes no pack file, no `webster-fragment-pack-read.md` exists, and the claims and tests pin the inline layout.
The board entry asks for the inline layout, so the code is right and the summary is stale from an earlier commit.
The summary is a claim checked against the diff, and the claim fails.

### [LOW] Unneeded move of the prompt path computation

**Location:** `internal/websterengine/beginbatch.go:561`
**Issue:** `promptPath` is computed before `buildKickstartPack` and `RenderForkPrompt`, but nothing between there and its old position uses it; it is still used only by `os.WriteFile` and the result.
The move came from the earlier design where the prompt named the pack file path.
It changes nothing in behavior, but no board-entry line licenses the diff hunk.

## Notes on the focus questions

- The prompt with a pack ends with the pack section, which opens with the one-turn read instruction followed by the cards inline.
  The `{{.kickstart_pack}}` marker is the last token of `webster-body-implementer.md`, and `claims_test.go` pins it as `slot-busy exit.{{.kickstart_pack}}\n`.
  No pack file, pack path parameter or pointer line exists.
- The batch-report section stays above the pack section and still names the report as the fork's last action.
- A render without a pack is unchanged, because an empty marker fill leaves the file's trailing newline as before.
  The recovery prompt takes no pack.
  An all-empty pack becomes nil in `buildKickstartPack`, logs `kickstart_pack=empty`, and has a test row.
- `doc.go` and the tests match the layout the code produces.
  No `.gitattributes` or test entry refers to a removed fragment.
