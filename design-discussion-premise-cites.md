# Discussion premises cite the glyph they rest on; the gate resolves them

`discussion-premise-cites` · Note · enhancement, loom, quarry, stencil

## Seen (Discussion bench, 2026-10-11)

Across the Discussion bench arms, most reference findings a draft repeated were reading errors: the code said otherwise and neither the writer nor any twill fork opened it (`classifyDeadlineExpiry`, `contractFileStatus`, `ownTrackedPaths`, go/build's GOOS rules).
Twill opus[medium] missed 4 reference findings, 3 of them reading errors; solo opus[high] missed 7, 5 of them reading errors.
Twill forks inherit the writer's reading, so they share its gaps.
No role is made to read, and nothing checks that a stated premise rests on code someone opened.

## Idea

- Every factual premise in the decision record about today's code names the symbol it rests on, as a glyph.
- The Discussion gate (`lyx loom validate-discussion`) resolves each named glyph with `lyx quarry resolve` and flags a premise with no glyph or with one that does not resolve.
- Reading cannot be forced, but a cited premise is checkable, and a premise like "`rusageStats` is split into per-OS files" fails when the cited files do not exist.

## Open

- Which sections of the record count as premises, so the rule does not reach design text.
- Whether the gate refuses or only warns.

## Operator decision (2026-10-15)

The Discussion writer does prose exploration and does not use quarry or glyphs; glyphs come in only at Plan.
So the idea above, glyph cites in the decision record resolved by the Discussion gate, conflicts with that decision as written.
What stays open is the finding itself: Discussion premises about today's code rest on reading nobody checks.
