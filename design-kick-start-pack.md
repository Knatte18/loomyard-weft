# Kick-start pack for Burler: pre-resolved glyph spans before the explore phase

`kick-start-pack` · Note · feature, burler, quarry

Read the closed GitHub issues #227 (implementer prompt) and #228 (Burler, with the operator's architecture-correction comment: inject once before the explore phase so every fork inherits it). Measured basis in quarry's `results/2026-09-06-kickstart` (`bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md`, commit `d2e492d`).

## Scope left here: Burler

The Webster implementer pack moved into task `plan-glyph-tuning` (2026-10-11), which also makes the pack switchable by config.
What stays here is the Burler side from #228: a pack of the reviewed glyphs injected once before the reviewer's explore phase, so every lens fork inherits it.
Take it up once the Webster pack has run with and without the pack and shown whether it helps.
