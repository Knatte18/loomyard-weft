```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the round-3 changes to the decision record and support log agree with the text around them.

- **MEDIUM**: the reworded D3 acceptance bullet ("an account whose weight was lowered, and a re-added account whose kept count is above that value, has its count lowered to it") contradicts D3's counter paragraph. The paragraph sets the count "whether that raises or lowers it". A lowered weight does not always lower the count. Take an account with count 2 and its weight cut from 2 to 1, while the others' lowest count over weight is 5: the rule sets it to 5 × 1 = 5, so the count rises. The criterion should say the count is "set to that value" and must not claim it is "lowered".
- **LOW**: in D2's fresh-launch bullet, the sentence the fix left untouched, "so neither `internal/shuttleengine` nor the claude engine reads reed's or orch's records", still names only reed and orch. The rewritten rule now covers more owners: sampler, PR-Review, gate fixer, webster and treadle. That sentence was not updated to match. It should name the owners as a group, for example "reads any owner module's records".
- **Outside this lens (fix-claims)**: the F1 rule says "every record under the worktree's anchor". Whether the sampler row's record and webster's state really sit under the worktree's anchor, and not under the hub board, was not checked.

I made no further tool calls.
