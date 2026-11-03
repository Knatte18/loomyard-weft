```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 fixes agree with the sections they touch or depend on, and find any change whose matching text elsewhere was left unchanged.

**1 finding (LOW)**

- **Severity:** LOW
- **Changed text:** `support-log.md`, Turn 4, the D10 item now reads "fixed in review round 1. An older binary's rename reaches that state, so D10 refuses it with `lym loom decision store` as the way forward."
- **Summary:** That item now refers to a review round, but the same log's `## Review rounds` section still says `_No rounds yet._`. The round-1 changes that touch the log are the Q11 rewrite and this item, and neither is listed there.
- **Evidence:** `support-log.md` keeps `## Review rounds` with `_No rounds yet._` unchanged, while the Turn 4 item and the Q11 rewrite record round-1 outcomes.

**Checked and consistent**
- D6, D8 and D9 now agree on the goto-past case.
- D2's Bound names D6, D8 and D9.
- D2's done-entry refusal matches the D8 and D9 wording.
- The D3 merge sentence matches the new D4 merge bullet and the D4 criterion.
- The rename move in the Notes line matches D4.
- The widened D1 criterion matches D1 and Q11.
- The Goal now says "refreshes", which matches D8's new refresh bound.
- The D10 refusal matches the Auto-mode older-binary line.
- The new D2 "flag without a file" sentence matches D4's crash sentence and the Auto-mode line.

**Outside this lens (for fix-way-forward):** D4's crash sentence and the Auto-mode line both send recovery to `lym loom decision store`. D9 needs a live run past its approval and refuses a done entry. So an entry whose run was cancelled or has landed, then renamed, has no route that re-stores its record.

**Further tool calls:** none.
