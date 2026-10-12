# Plan gate: receiver-typed delete-before-reference; plan review validates its fixes

`plan-gate-fixes` · Task · bug, loom, quarry, stencil · running · Plan-Write

Two Plan-stage bugs from task `burler-split-budget` that cost Plan-Write time and Plan-Review rounds.

## 1. `delete-before-reference` matches deleted methods by bare name (issue #562)

The plan gate's `delete-before-reference` check matches a deleted method glyph's member name as a bare word.
Deleting `probingBurlerRunner.Run` or `shedfake.BurlerRunner.Run` flagged every later card whose edit code held any `.Run(` call, `doc.go` prose included, and `BurlerProducer.Call` drew a match on an unrelated `.Call` in `internal/proc/namedjob_windows.go`.
About 220 blocking findings came from a handful of deletes.

Dropping the method glyphs does not help: `caller-uncovered` then blocks on the method bodies.
The only admissible plan shape was moving every deletion into the last card and deleting a test helper type by listing its whole file instead of its methods, which loses the `caller-uncovered` check on those methods.
That forces an unnatural card order: retiring the old producer right after the switch is refused.
Plan-Write ran about 30 minutes and 192k output tokens, much of it on repeated `lyx loom validate-plan` passes.

Expected: match a deleted method by receiver type, counting only selector expressions whose receiver resolves to the deleted type when a type load answers, as `caller-uncovered` already does.

## 2. Plan-review fixes are proposed without validating them; fix glyphs name the wrong function (issue #563)

A plan-review round proposed a fix whose method Delete glyphs the plan gate refuses (the bug above).
Finding that out took two validator runs, after the fixer had already applied it.
The rubric says a reviewer proposes only fixes the checks admit, but nothing makes the reviewer verify that.
A reviewer that runs `lyx loom validate-plan` against its proposed target changes would catch this before the fixer does.

Separately, another finding's proposed glyphs named the function after or near each cited comment rather than the function that encloses it, so each glyph had to be located again.

Expected: the plan-review prompt directs the reviewer to validate proposed target changes, and to name the enclosing function for glyphs tied to a comment.

Provenance: task `burler-split-budget`; notes `Plan-Write.md`, `burler-plan-r1.md`.
