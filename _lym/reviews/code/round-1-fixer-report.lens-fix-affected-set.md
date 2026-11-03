```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that F1's fix covers every site under its key (the "stencil's static prose" scope in the Test economy rubric).

Findings:

- LOW — `contracts/stencils/loom/loom-block-plan-format.md:16` still reads "A stencil's static prose is not behavior a test covers." This is the same narrow wording F1 widened in the code rubric. The reviewer rejected widening it as out of scope, and the fixer's change set does not touch it, so the stencils now disagree on scope. It is a reader-noticeable sibling mismatch but was explicitly rejected in the review, so it is not a fix defect.

The F1 fix covers both sites in `loom-rubric-code-review.md` (lines 109 and 110). No other "stencil's static prose" occurrence remains in that file or in `loom-rubric-plan-review.md`.

Calls:
1. `grep -rn "static prose" contracts/stencils/loom/`: re-ran F1's search across the stencil directory to find any remaining narrow-wording sites. My context lacked the other stencil files.
