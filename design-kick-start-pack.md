# Kick-start pack: pre-resolved glyph spans for implementer and Burler

`kick-start-pack` · Note · feature, webster, quarry

Read the closed GitHub issues #227 (implementer prompt) and #228 (Burler, with the operator's architecture-correction comment: inject once before the explore phase so every fork inherits it). Measured basis in quarry's `results/2026-09-06-kickstart` (`bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md`, commit `d2e492d`).

## Webster (operator, 2026-10-10)

The Webster implementer is the natural first user: its batch's cards already list the glyphs it touches (`Edit:`, `Uses:`, `Delete:`), and it has nothing to explore.
The pack is built per batch and goes only into that batch's implementer fork prompt, never into Merriam's context, which would bloat with every batch.
