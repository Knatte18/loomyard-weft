```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling decisions, criteria and cross-references in the decision record.

- **LOW**: D8's held path, the fallback in "with neither, it is the parent's head, as a held run records it today", combined with D6's new "the landed size and the empty-landing check measure from that commit's first parent".
  Summary: the held path says its records follow "as D6 says", so a held run that falls back to the parent's head takes the landed size and the empty-landing verdict from the first parent of whatever commit heads the parent. That can be another run's landing.
  Evidence: D6 now ties both measures to the run's own landing commit. Today's held run measures from `parentHeadMerged`, which `keepParentHead` sets only during the merge (`internal/landingshed/finalize.go:996-1022`), and the held path no longer runs that merge. The fallback needs to say whether it records a size and an empty-landing verdict at all.

Checked and consistent:
- D3's leader test.
- D2's "parent is read first (D8)".
- The `landed` bullet now sends the run to the held path.
- The `failed` and `conflict` bullets lose to the held-work test, whatever the entry says.
- Criterion line 239 and the rewritten D8 criterion.
- The new D6 criterion against the Constraints line.
- D6's rationale ("never ahead … by a commit that is not an owed landing tip").
- The plan-writer note naming `Deps.RecordLanding` as the dobby's record.
- Support-log Q10.

I made no further tool calls.
