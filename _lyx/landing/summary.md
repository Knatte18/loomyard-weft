# Webster fork prompt: end on the pack read line, pack in its own file

The Webster implementer fork prompt now ends with a one-line `## Start here` section that sends the fork to its kick-start pack file.
The one-turn read instruction and the pack moved into that file, so the fork opens it last and acts on it first.
The audit of fork reads found many forks still reading serially in their first work turn; the end of a prompt is what an agent acts on first.

## What changes

- `begin-batch` writes `<batch>-pack.md` beside the fork prompt file when the batch has a pack, before it writes the prompt, so the prompt never names a missing file.
- The pack file opens with the instruction to read, in one parallel turn and before any edit, every card file and every listed span, then the uncovered target and `Uses:` files and the located unresolved glyphs' files.
  Then it holds one subsection per card.
- The FRESH-READ rule at the top of the prompt no longer carries the read instruction; it points at the pack file.
- The batch-report section keeps its text and still names the report as the fork's last action; only its place moves above the new final section.
- A batch none of whose cards has a resolved or unresolved entry now builds no pack, logs `kickstart_pack=empty`, and renders as with the pack off.
- Without a pack, no pack file is written and the prompt reads as before, byte for byte; recovery prompts never take a pack.
- `RenderForkPrompt` takes the pack file's path and returns the pack file's content alongside the prompt.
- New fragment `webster-fragment-pack-read.md` (registered in `contracts/stencils/stencils.go` and `.gitattributes`); the pack fragment banners now name the pack file.
- `internal/websterengine/doc.go` describes the pack file and the empty-pack case; the stencil layout tests and the begin-batch tests are updated to match.

## Design decision

The board entry asks for the instruction and the pack inline as the prompt's last section.
This change departs from it: the pack lives in its own file and the prompt ends with a single read line.
That departure follows the operator's decision, recorded in commit d8d1c0956 ("restore the kick-start pack's own file, per the operator's decision").
The prompt's last line is then one instruction, and the pack's size no longer pushes the batch-report section away from the prompt's end.

## To check by hand

- Not run: the measurement the board entry names.
  Re-run the glyph-audit scripts on runs after this lands: the share of forks with three or more parallel reads in the first work turn, and exploration calls per card.
