# Kick-start pack: pre-resolved glyph spans for implementer and Burler

`kick-start-pack` · Note · feature, webster, quarry

Read the closed GitHub issues #227 (implementer prompt) and #228 (Burler, with the operator's architecture-correction comment: inject once before the explore phase so every fork inherits it). Measured basis in quarry's `results/2026-09-06-kickstart` (`bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md`, commit `d2e492d`).

## Webster (operator, 2026-10-10)

The Webster implementer is the natural first user: its batch's cards already list the glyphs it touches (`Edit:`, `Uses:`, `Delete:`), and it has nothing to explore.
The pack is built per batch and goes only into that batch's implementer fork prompt, never into Merriam's context, which would bloat with every batch.

## Why mechanical: the stencil instruction is ignored (2026-10-11)

The follow-up count in `glyph-practice-audit` (2026-10-11) measured the implementer stencil line from a28e79d9c that tells each fork to resolve its card's glyphs with `lyx quarry resolve` before any grep.
Of 51 Webster implementer forks on the new stencil, 49 had glyphs on their cards and only 16 called quarry at all; use fell from 12 of 34 forks on 2026-10-10 to 4 of 17 on 2026-10-11, and forks that did call it grepped as much as the rest.

Operator, 2026-10-11:

- The fork does not call `lyx quarry resolve` itself; the Go code resolves the batch's glyphs as part of the fork's startup and injects the result into the prompt or the file the fork reads.
- The fork must still open and read the code files the resolved spans point to; the pack tells it where to read, it does not replace reading.
