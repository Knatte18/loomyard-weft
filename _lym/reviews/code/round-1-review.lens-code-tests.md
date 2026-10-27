```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens over the amendment, which is limited to the `docs/overview.md` overlap.

**Per behaviour, the test that pins it:**
- The creel tree rows and the module bullet in `docs/overview.md` (`hook-read` verb, flag files, once-delivery sentence) are static doc prose. They are exempt from "a behavior the diff adds with no coverage" in the rubric's Test economy, and no test pins them. Verdict: none needed.
- The parent's darn-recipe and regression-row sentences are static prose. No test pins them, and they are not this task's behaviour.
- The amendment adds no code and no test. The merge changed no test files, so nothing is redundant or per-helper, and no review-fix test lacks a coverage gap.
- Earlier fix commits F1–F5 are settled and out of amendment scope. Nothing about them is re-judged here.

**Findings:** none.

**Tool calls:** none made. The overlap is documentation only, and the context already held the diff of `docs/overview.md`.
