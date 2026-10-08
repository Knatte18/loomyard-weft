# Audit how plans use glyphs in practice, then tune the plan stencil

`glyph-practice-audit` · Note · enhancement, loom, stencil

## Why

`plan-glyph-precision` makes plan cards name the members they change as glyphs, gated by the plan validators, and its committed test proves the glyph chain over a fixture module.
That proves the machinery, not that plan writers use it well.
The operator (2026-10-08) doubts a writer can pick the right glyphs from its own reasoning, and expects the gate to cost Plan rounds and time.
Kick-start packs (`kick-start-pack`, `enclose-kickstart`) build on plans whose glyphs resolve, so they wait until practice shows that.

## Proposal

Audit the first runs after `plan-glyph-precision` lands from their logs and transcripts:

- the share of `Edit:` targets that are member glyphs rather than file paths or self glyphs;
- the Plan rounds and Plan wall time spent on glyph findings (unresolved glyphs, redundancy, caller coverage, the re-sign arrow), against the Plan time of earlier runs;
- `plan:` handles declared, bound at record-batch, and failed;
- members a card edited that its glyphs did not name, and glyphs that named the wrong member, read from the batch diffs.

From what the audit shows, tune the plan stencil: where writers guess, show them how to resolve a glyph with quarry before they write it, so the gate rejects less.
Kick-start work starts only once the audit shows plans name resolvable members in practice.
