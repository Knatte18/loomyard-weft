# Glyphs end to end: test members as glyphs, traced plan: binds, a switchable Webster kick-start pack

`plan-glyph-tuning` · Task · enhancement, loom, stencil, webster, quarry · awaiting · PR-Gate

## Basis

The glyph practice audit (2026-10-10, follow-up 2026-10-11) found the plan side sound: about 60% of plan targets are glyphs, glyph findings are rare and fixed in one pass, and the gate costs no extra Plan rounds or time.
The execution side is not: implementer forks ignore the card's glyphs.
This task makes glyphs work end to end, from the plan card to the Webster fork: two plan-side items and the Webster kick-start pack.

## What to do

- **Test members as glyphs.**
  `contracts/stencils/loom/loom-template-plan.md` still lists "a test file the card adds tests to" as a plain-path target under `**Edit:**`.
  Writers already name existing tests as glyphs on their own (140 `#Test*` glyphs under `Edit:` in the follow-up sample, against 1 before), and about two thirds of test-file edits change an existing test, fake or helper a glyph could name.
  Make the stencil and the plan review rubric say so: a card names the existing test, fake or helper it changes as a glyph (`internal/x#TestFoo`, `internal/x#fakeClock`), keeps a plain path only for a test file with no member change, and declares a new test as a `plan:` handle or lists its new file under `**Create:**`.
- **Make `plan:` bind outcomes observable.**
  Webster's record-batch binds each `plan:` handle, but no trace line records whether it bound or failed, so the audit could not count them.
  Log one trace line per handle at record-batch: bound or failed, with the reason.

## Webster kick-start pack

Background: closed GitHub issues #227 (implementer prompt) and #228 (Burler; the operator's comment there: inject once before the explore phase so every fork inherits it).
Measured basis in quarry's `bench/loomyard-eval/ladder/results/2026-09-06-kickstart/conclusion.md` (commit `d2e492d`).
The Burler side stays in note `kick-start-pack`.

### Why mechanical: the stencil instruction is ignored

The glyph audit's follow-up (2026-10-11) measured the implementer stencil line from a28e79d9c that tells each fork to resolve its card's glyphs with `lyx quarry resolve` before any grep.
Of 51 Webster implementer forks on the new stencil, 49 had glyphs on their cards and only 16 called quarry at all; use fell from 12 of 34 forks on 2026-10-10 to 4 of 17 on 2026-10-11, and forks that did call it grepped as much as the rest.

### Wanted

- The Webster implementer is the first user: its batch's cards already list the glyphs it touches (`Edit:`, `Uses:`, `Delete:`), and it has nothing to explore.
- The Go code resolves the batch's glyphs as part of the fork's startup and injects the result (file, line span, signature per glyph) into the batch's implementer fork prompt or the file the fork reads; the fork does not call `lyx quarry resolve` itself (operator, 2026-10-11).
- The pack is built per batch and goes only into that batch's fork, never into Merriam's context, which would bloat with every batch (operator, 2026-10-10).
- The fork must still open and read the code files the resolved spans point to: the pack tells it where to read, it does not replace reading (operator, 2026-10-11).
- A glyph that does not resolve (a `plan:` handle not yet bound, a member deleted) is listed as unresolved in the pack, never an error.
- **The pack can be switched on and off with one config key** (operator, 2026-10-11): it is not proven that the pack helps, so it must be easy to run with and without it and compare, and to turn it off if it gives nothing. Off gives today's prompt unchanged.
- Once it runs, repeat the audit's execution-time count (grep-for-member calls and whole-file reads per fork) with the pack on and off.
