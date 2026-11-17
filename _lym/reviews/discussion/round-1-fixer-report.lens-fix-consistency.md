```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the fixer's changes to the decision record agree with the sections around them.

**[LOW]** The Constraints line `PATTERN-refusal-way-forward` still says only "every new refusal of the review and fix checks".
- **Summary:** That line was not updated after the round checks moved into their own gate entry, so it does not cover that entry's refusals.
- **Evidence:** D3 now reads "These round checks, with D4's bounds, form a round-check gate entry … separate from the review check `ParseCurrentReview` runs". The prior, carries, severity, location-path, BLOCKING, file-bound and `origin: parent` refusals all live in that entry now, but the constraint still names only "the review and fix checks".

**[LOW]** D3's new rule that a `decision`-class finding the parent answered "takes `resolved` with an `answer` citing that answer".
- **Summary:** The round-check entry's refusal list and the D3 acceptance rows do not cover this new use of `answer`. A `resolved` decision finding with no `answer` is neither refused nor named as allowed, so the rule disagrees with the check's own list.
- **Evidence:** The D3 bullets define `answer` as required only for "`open` on a disputed finding". The entry's list covers only "lacks `answer` on an open disputed one". The acceptance row says only that it "takes `resolved` and is not carried".

**[LOW]** The Review acceptance row says the review-file format states "D3's `prior` and `carries` keys and D4's bound, including that a fix's regression … is located at the changed line".
- **Summary:** This row lists what the format must state, but it leaves out two rules the reviewer must now follow: a carried finding keeps its earlier location path, and an answered decision finding takes `resolved` with `answer`.
- **Evidence:** D3 adds "keeps its earlier finding's location path" and "takes `resolved` with an `answer` citing that answer". D2 says "The rules the reviewer follows on the block live in `contracts/stencils/burler/burler-step-review.md` and the review-file format".

Tool calls made: none.
