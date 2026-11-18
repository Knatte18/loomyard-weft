```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with the sibling decisions, criteria and cross-references it touches (fix-consistency lens).

**MEDIUM: a re-run merge verb's own step 1 would abort the merge that D5 says it refuses and never aborts.**
- Changed text: D5, "A merge verb run again over either refuses with today's already-in-progress guard and never aborts the merge".
- Evidence: in the same decision, step 1 says every merge verb "commits the receiving worktree's uncommitted records (D4)". D5's guards paragraph says "Fabric's own dirtiness guards (the merge verbs', …) commit the worktree's records first in the same way". D4, as changed, has a records commit that finds a fabric merge stopped before step 3 abort that merge. The change never says the already-in-progress check runs before step 1 or before the guards' records commit. So in the pre-step-3 case the two sentences contradict each other. Today the record check comes first only inside `recheckMergePreconditionsUnderLock` (`internal/fabricengine/merge.go:66-73`), and the record never ties the guards' records commit to that ordering.
- Fix: state that a merge verb runs its already-in-progress refusal before step 1 and before its guards commit records.

**LOW: "either" in the D4 acceptance row covers three places.**
- Changed text: D4 acceptance row, "leaving the status file in HEAD, the worktree and origin and no source records in either".
- Evidence: the sentence lists three places, so "either" doesn't say which two it means. It should read "in any of them".

No further tool calls were made.
