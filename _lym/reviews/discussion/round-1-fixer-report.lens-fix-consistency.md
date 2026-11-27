```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that the changed text in `decision-record.md` and `support-log.md` agrees with its sibling decisions, criteria and cross-references.

1. MEDIUM. **Changed text:** D1, "When the probe fails, the window stays where KWin opened it, unplaced and unrecorded", and the D1 acceptance criterion "leaves the `--separate` window unplaced and unrecorded".
   **Summary:** the window cannot be unrecorded, because the child records it before any query runs the probe.
   **Evidence:** D9 says "A placement first records the new window's process id and title, before any wait." D10 says "The child records the window (D9), waits for it to appear in KWin…", and the probe runs in the first query script (D1, D3 step 1), which comes after that write. So a window whose probe fails already has an unplaced entry. Its children, whose parent wait in D8 waits for that entry to become placed, then wait the full 180 s.

2. LOW. **Changed text:** D8, "Past the wait the parent's remembered desktop is the anchor", and the D8 acceptance criterion "past the wait the parent's remembered desktop, then the current one, is the anchor".
   **Summary:** D9 now holds two desktops per entry, and "remembered desktop" no longer says which one.
   **Evidence:** D9 gives each entry "the window's home desktop … [and] the last desktop the window was placed on or seen on". D8's sentence just before it already says "the entry's last desktop" for the placed case, but this twin sentence kept the old wording.

3. LOW. **Changed text:** D9, "Every write of the places file is one `state.UpdateJSON` read-modify-write that changes only its own window's entry, and, on a write after an apply, the entries the rules below change."
   **Summary:** the "only" claim leaves out the drop of entries for removed worktrees, which also rewrites other windows' entries.
   **Evidence:** D9's later paragraph, "Entries for a worktree the hub no longer has are dropped at that write", sits outside the bullet list that "the rules below" refers to, so the scope of the guarantee does not say whether it covers the drop.

No further tool calls were made; everything above comes from the record as changed in this context.
