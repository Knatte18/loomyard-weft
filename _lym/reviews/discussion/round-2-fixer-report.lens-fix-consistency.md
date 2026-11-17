```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix set agrees with its sibling decisions, criteria and cross-references, and look for a twin edit that was missed.

- LOW: `decision-record.md` D5, first bullet, which now reads "runs from anywhere inside the worktree's anchored directory or its records sibling" (F6). Its twin acceptance criterion was not narrowed to match. It still reads "`lym board open` run from a subdirectory of a worktree with `board.html` present hands the opener seam the absolute path…". For a subpath-anchored worktree, a subdirectory outside the anchor is refused by `lymcwd.ResolveWithin` with `ErrCwdOutsideAnchor`, which the review's F6 issue cites. Fix: change the criterion to say "a subdirectory of the worktree's anchored directory".
- LOW: `support-log.md`, Question ledger Q8, a sibling of the D7 removal (F1). It still reads "auto-pick: collapsible Tasks, Notes and Done per project (D7); raised under Open risks for the parent". This points at a decision and an Open risks line the fix removed, and it gives no outcome, so a reader of the ledger never learns that the pick was dropped. The new Rejected-alternatives line and the Round 2 line record the drop, but the ledger row was left as it was. Fix: add to Q8 that it was dropped in round 2 after the parent's answer.

I made no further tool calls; the context already held every file in its changed form.
