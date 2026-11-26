```yaml
lens: fix-code-correctness
outcome: reported
```

Task: checked the code the fix changed (fix-code-correctness lens).

**BLOCKING** — `contracts/stencils/drift/drift-template-judge.md` Step 2, the new carried-check paragraph (also the new `doc.go` sentence): a `none` ruling that depends on another file is reused by the file's own key, so removing the carrying check later passes unjudged.

Evidence, one concrete run:
1. Pass 1: test file A is deleted, and its assertion is added to B, a file that did not hold it at the review base. Under the new rule A is `none` ("B carries it") and B is `none`. The row is Done.
2. A later commit, such as a Code-Burler-Fix before `Drift-Recheck`, drops that assertion from B, or reverts B.
3. `listFiles` (`loosened.go`, the `slices.IndexFunc` clause) reuses A's ruling: its diff and the trailers are unchanged, and it holds no uncaused check, so the range key is ignored.
4. B is either relisted with a diff against the base that never shows the assertion (it was added and removed within the range), so there is nothing to rule loosened, or B leaves the range altogether.

The check is gone from both files and the pass is Done, with no trailer and no halt. Before this change A was ruled `loosened` on its own diff, so the F1 fix opened this path. The same applies to a caused or `loosened` ruling on A that left one carried check out of its `checks`.

A closing fix needs the engine to know that a ruling relied on another file, for example a judge-reported marker that makes the ruling reusable only under an unchanged range key. Reusing every ruling that way would contradict the F2 pin that an unrelated `none` ruling is reused.

No other findings:
- **`rangeRulingKey`:** its one caller passes `files` only, and `trailers` is still used by `fileRulingKey`. A trailer-only commit still relists every file through the file key.
- **New test row, defect in place:**
  - With reuse by the file's own key alone, A is not relisted and `wantFiles` fails.
  - With a paths-only range key, `wantFiles` fails the same way.
  - With the `!holdsUncausedCheck` clause dropped, the untouched file is listed and `wantFiles` fails.
- **Halt text and comments:** the new halt substring matches the test, and the reworded comments agree with the code.

Further tool calls: none.
