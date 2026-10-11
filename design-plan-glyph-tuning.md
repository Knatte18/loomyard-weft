# Plan glyphs: test members as glyphs, trace plan: handle binds

`plan-glyph-tuning` · Note · enhancement, loom, stencil, webster

## Basis

The glyph practice audit (2026-10-10, follow-up 2026-10-11) found the plan side sound: about 60% of plan targets are glyphs, glyph findings are rare and fixed in one pass, and the gate costs no extra Plan rounds or time.
The execution-side gap (implementer forks ignoring the card's glyphs) moved to `kick-start-pack`.
Two plan-side items remain.

## What to do

- **Test members as glyphs.**
  `contracts/stencils/loom/loom-template-plan.md` still lists "a test file the card adds tests to" as a plain-path target under `**Edit:**`.
  Writers already name existing tests as glyphs on their own (140 `#Test*` glyphs under `Edit:` in the follow-up sample, against 1 before), and about two thirds of test-file edits change an existing test, fake or helper a glyph could name.
  Make the stencil and the plan review rubric say so: a card names the existing test, fake or helper it changes as a glyph (`internal/x#TestFoo`, `internal/x#fakeClock`), keeps a plain path only for a test file with no member change, and declares a new test as a `plan:` handle or lists its new file under `**Create:**`.
- **Make `plan:` bind outcomes observable.**
  Webster's record-batch binds each `plan:` handle, but no trace line records whether it bound or failed, so the audit could not count them.
  Log one trace line per handle at record-batch: bound or failed, with the reason.
