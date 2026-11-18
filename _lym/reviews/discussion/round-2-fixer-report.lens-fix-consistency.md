```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether the F12 change set agrees with the sibling text it touches, and find any twin edit that was missed.

1. LOW. D5 (`decision-record.md`, D5 body): "It is the most findings PR-Review fixes in place when none is BLOCKING."
   Since the F12 change, the threshold caps the running fix-here count between two send-backs, or before the first, and no longer caps a single set.
   D4's send-back bullet now says "the set plus the fix-here count below exceeds `loom.pr_review_small_max`", and D4's Bound says "at most `pr_review_small_max` findings are fixed in place between two send-backs, or before the first".
   D5's sentence reads as a cap on each set, so a reader would notice that it disagrees with those two lines.
   One way to bring it in line: "the most findings PR-Review fixes in place between two send-backs, or before the first, when none is BLOCKING."

The other changes agree with their siblings:
- The rewritten D4 sentences: the clean-tree wake clears the count, and only that reroute clears it.
- The dirty-tree resume criterion, and the reworded D4 criteria on the count across a send-back and on a re-step.
- D2's Bound, now "between send-backs".
- D4's Rationale, which still speaks of the target segment's fresh episode.
- The Round 2 entry in the support log.

The Round 1 self-check line in the support log ("the count clears per episode") is a record of that round, so it is out of scope.

Tool calls: none; my context already held the changed files.
