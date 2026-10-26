```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 and F2 proposals, both fixed as proposed (doc.go line 11 and the Validate comment in validate.go).

no findings

Both reworded texts match the code. `patternRuleText` strips a trailing background link and then a trailing ` (test)`, which is what the new doc.go wording says. The check applies to `adds` and `changes` bullets and skips `removes`. The finding detail reads "over the %d cap of a PATTERN.md entry's rule text", the same phrase the new Validate comment uses. No sibling text still describes the old whole-line cap.

Calls made:
1. `sed -n 140,190p internal/discussionparser/patternchanges.go` plus a grep for `patternEntryTextCap`: my context lacked the check's code, which I needed to test the new wording against what it measures.
