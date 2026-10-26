```yaml
lens: fix-affected-set
outcome: reported
```

Findings: no findings.

- F1 (doc.go:11) and F2 (validate.go:61) are the two sites where the review says the package still describes the old whole-line cap.
- A grep of non-test files in `internal/discussionparser/` for "overview line", "entry line", "line cap" and "whole-line" finds nothing stale.
- `patternchanges.go:159` says "the part of an entry line the cap measures". That is accurate for the rule-text cap.
- `relevant.go:13` uses "entry line" for the relevant list, which is unrelated.

I did not search outside `discussionparser`. The review already verified no remaining `MaxEntryLineChars`, `patternEntryLineCap` or 360 cap references there.

Tool calls made:
1. `grep` over `internal/discussionparser/` non-test files, for stale wording that F1 and F2 (the set of sites still describing the old cap) should have covered. My context lacked the other files in the package.
