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
- members a card edited that its glyphs did not name, and glyphs that named the wrong member, read from the batch diffs;
- how the plan writer arrives at each glyph, read from the Plan-Write transcript: the `lyx quarry` calls it made, the greps and file reads around them, and how many tries a glyph took before it resolved (operator, 2026-10-08: the search may be mostly trial and error, and its cost belongs in the audit);
- whether implementer and Burler forks use the card's glyphs at execution time (a `lyx quarry` lookup) or fall back to grep and whole-file reads.

## First data points (2026-10-08)

- `shuttle-send`: 93 of 139 `Edit:`/`Create:` targets are glyphs; every plain path is a test, `doc.go`, `template.yaml` or markdown file; one self-check round of 10 `glyph-not-found` (plan-created members named without `plan:`), fixed in one pass; Plan-Write about 13 minutes, Plan-Review one round.
- `review-lens-fan`: 34 of 50 targets are glyphs, again with no plain path to non-test Go source.

The plan stencil already tells writers to copy glyphs from `lyx quarry` answers and never spell one; plans before this task still named mostly file paths, because a path was always a valid target.
The audit's main question is whether the share of plain paths falls once `plan-glyph-precision`'s member-glyph rules and rubric flag are in place.
From what it shows, tune the plan stencil and rubric where writers still fall back to paths or take many gate rounds.
Kick-start work starts only once the audit shows plans name resolvable members in practice.
