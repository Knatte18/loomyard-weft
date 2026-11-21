```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the F1–F6 change set. I made no further tool calls.

**Findings**

1. LOW. Changed text: the `mergeOnGitHub` doc comment in `internal/landingshed/finalize_pr.go`, the line "the merge is retried while that state reports the pull request can merge, scheduled only while the window has not passed."
   - The text after `//` is about 121 characters, over the prose rule's limit of under 120.
   - It should be split at the comma into two lines.
   - The new line in the `mergeRefusalReason` doc, "except that a refusal outlasting the window while GitHub reports the pull request can merge names the resume alone.", is about 118 characters. That is within the limit but borderline.

2. LOW. Changed text: `_lym/landing/summary.md`, sections "What changes" and "Tests".
   - The summary was only edited under "Merge danger". Its other sections no longer match the code.
   - "Tests" says `TestFinalize_PullRequest` "gains three rows". The fix added a fourth, `AFailedMergeStateReadAfterANotMergeableRefusalHaltsAndMergesNoSecondTime`.
   - "Tests" also says the `unknown` row "halts naming it". That row now also asserts no approval and no cancel advice.
   - "What changes" does not mention the new halt for an uncomputed mergeability. That halt names waiting for GitHub and the resume, with no approval or cancel, and a beam's version also names `lym beam pr status`.
   - "Design decisions" says the poll can add up to one more minute, which agrees with the F3 doc text.

**Checked and consistent**

- `doc.go` line 643 still says the halt "also names a second person's approval or a settled check or conflict, and `lym batten cancel`". The reworded sentence before it now limits that to "any other cause", so the two agree.
- The new `doc.go` sentence on the uncomputed case matches the `MergeabilityUncomputed` branch. That branch needs the state open, `unknown`, and exactly one cause. A draft or a head mismatch adds a cause, so those states still get the old advice.
- The beam exception in the `doc.go` sentence matches the `mergeRefusalReason` doc and its early return.
- The `RefusalOther` and `prMergeRetryWindow` docs match the retry and read logic in `mergeOnGitHub`.

No tool calls were made.
