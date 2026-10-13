---
round: 1
focus:
  - "Scope against the board entry: the entry asks for the instruction and pack as the prompt's last section, while the diff writes the pack to `<batch>-pack.md` and ends the prompt with a read line. Settle whether the summary's 'operator's choice' claim is borne out by the diff and the repository, and whether the no-pack prompt is byte-identical to before."
  - "The empty-pack change (`kickstart_pack=empty`, no pack file): settle whether the entry licenses it, and whether every path that previously rendered a pack for a batch with no resolved or unresolved entry now agrees on the empty state, including the trace line and the recovery prompt."
  - "Stencil layout in `webster-body-implementer.md` and the fragments: settle that the `kickstart_pack` marker sits at the body's end, the batch-report section still names the report as the fork's last action, and the FRESH-READ pack and plain fragments carry no stale read instruction or dangling reference."
  - "Pack file lifecycle in begin-batch: settle that the file is written beside the fork prompt, that its path in the read line is the one the fork can open, and that write failure and cleanup are handled like the prompt file."
  - "Docs and test economy: settle that `internal/websterengine/doc.go` and any PATTERN or docs-overview text match the new layout, and that each new test names behavior no existing test covers (check the render tests against `TestBeginBatch_KickstartPack`); also check the test-budget file if a package passed its row."
---

Seed focus only; the reviewer still reads the whole committed diff against the board entry.
