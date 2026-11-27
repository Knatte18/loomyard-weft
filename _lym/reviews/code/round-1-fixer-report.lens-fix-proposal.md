```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fix carried out (F2, F3, F4, F5, F7, F8, F9).

I found no case where a carried-out proposal makes code or tests wrong. The findings are two inaccuracies in the F7 trailer commit (`536d89d9d`):

1. **LOW.** The `TestNew_HappyPath` trailer says "the warp and weft fields are no longer read".
   - Other tests still read them: `fabricengine.WarpForTest(f)` was in the deleted merge-recovery test and `WeftForTest(f)` is in `TestMergeState_ResetMergeSides`.
   - What holds is that no check wants the two fields non-nil after construction.

2. **LOW.** The `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` trailer leaves out a removed assertion the named holder does not make.
   - The removed step also failed on `strings.Contains(err.Error(), "push code side")` ("want the code side untried").
   - `RefusingRecordsRemoteWithholdsTheCodeSide` checks the two wanted substrings, the unmoved code bare and the failed-push record, but has no such negative check.
   - The trailer reads as though the holder covers the whole step.

I checked that the F9 prunable case fails without the skip: with the directory gone, `committedMarker` → `showFile` → `ResolveSHA("HEAD")` returns an error, not "not found".

Tool calls:
- Read `committedMarker` and the scratch-link phrase table: neither was in context.
- Read `showFile`: needed to know whether a missing directory errors or reads as an absent marker.
