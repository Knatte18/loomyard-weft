# Plan-review fixes are proposed without running lyx loom validate-plan; fix glyphs name the wrong function

`plan-review-validate` · Note · bug, loom, stencil

Imported from [issue #563](https://github.com/Knatte18/loomyard/issues/563).

A plan-review round proposed a fix whose method Delete glyphs the plan gate refuses (see the `delete-before-reference` text-matching issue).
Finding that out took two validator runs, after the fixer had already applied it.
The rubric says a reviewer proposes only fixes the checks admit, but nothing makes the reviewer verify that.
A reviewer that runs `lyx loom validate-plan` against its proposed target changes would catch this before the fixer does.

Separately, another finding's proposed glyphs named the function after or near each cited comment rather than the function that encloses it, so each glyph had to be located again.

Expected: the plan-review prompt directs the reviewer to validate proposed target changes, and to name the enclosing function for glyphs tied to a comment.

Provenance: task `burler-split-budget`; note `burler-plan-r1.md`.
