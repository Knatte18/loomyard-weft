```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's Goal points and checked whether every point is built.

**Decisions.** The record has no `Added after Discussion` entries, so the decisions are the Goal points of the board entry, `decision-record.md:13`:
- **G1:** godocreflow processes blocks that hold headings or lists.
- **G2:** godocreflow respects the line-length limit and does not join lines past it.
- **G3** (the alternative to G1 and G2): stop pointing at godocreflow as a fix.
- **G0:** the tool no longer adds wrapped-break findings. This is the "Why abnormal" point at `decision-record.md:11`, which frames G1 and G2.

**Per change:**
- **C1: commentlint exports its rules.** `IsGenerated`, `IsDirective`, `IsProseLine`, `IsSemanticBreak` and `TextReachesLimit` are exported, `countedText` and `isExemptFromLength` are reworked, and `doc.go` is updated (`lint.go:84-92`, `lint.go:193-201`, `lint.go:269-283`, `linelength.go:22-66` in the diff, `doc.go:32-33`). Forced by C2–C4, which apply the lint's own rules instead of the regexes the tool deleted. The `doc.go` paragraph ("re-implements them") went stale because of that change. Traced.
- **C2: each paragraph is reflowed on its own.** `reflowGroupLines` and `reflowParagraph` (`reflow.go:169-207`) keep heading, list and code paragraphs as written and reflow the prose around them. This builds G1.
- **C3: breaks follow the lint's break rule.** `layoutParagraph`, `splitAtBreaks` and `joinRejectedBreaks` (`split.go:234-307`) keep written breaks and join rejected ones. This builds G0.
- **C4: the length limit replaces the column wrap.** `splitLongLine` (`split.go:309-348`), the removal of `wrapLongLine` and `-max-width`, `collectEdits` taking `commentlint.LineLengthLimit` (`main.go:64`), and the one-line trigger at `reflow.go:144`. This builds G2. A joined line that reaches the limit is re-split at an accepted break or a written one, and `splitLongLine` keeps the original wrap positions in `writtenBreaks`.
- **C5: long lines are named for a hand rewrite.** `reportLongLines`, the `edit.longLines` field and the closing message (`main.go:90-92`, `main.go:129-144`, `reflow.go:153-158`). Forced by C4: a line with no accepted or written break stays over the limit, and the run must name it. This is G2's remainder.
- **C6: the wider directive set.** Groups with `//lym:` or `//testtiming:` are now left alone. Forced by C1's reuse of `IsDirective`: it comes with the lint's directive set and is not a separate feature.
- **C7: `edit` field comments.** They moved from the end of the line to above it (`reflow.go:18-27`). Forced by the field change (`wrapped` became `longLines`) in the struct the diff touches.
- **C8: test changes.** The rows in `reflow_test.go`, and the removal of `TestWidthFallbackAppliesToOverwideAtomicLine` and `TestWrapLongLine` along with the code they tested. Forced by C2–C6.

**Per decision:**
- G0: C3, with C1.
- G1: C2.
- G2: C4 and C5, with C1.
- G3: no change, and none is needed. The Goal offers it only as an alternative to G1 and G2, and the diff builds G1 and G2. A grep in my context found no doc or stencil that points at godocreflow as a fix, so nothing is left wrong.

**Findings:** none. Every change traces to a Goal point or is forced by a traced change, and every Goal point the diff must build is built.

Tool calls: none. The context already held the diff, the record, the board entry and the reference greps.
