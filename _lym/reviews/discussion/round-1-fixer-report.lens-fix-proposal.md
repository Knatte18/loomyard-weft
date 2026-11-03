```yaml
lens: fix-proposal
outcome: reported
```

Task: check the fixes made as the reviewer proposed (F1–F4) by building cases where a proposal goes wrong.

**[MEDIUM] The new D2 criterion conflicts with replacement**
- **Changed text:** `decision-record.md`, Acceptance criteria, the added D2 bullet: "with two overview windows and no live agent, a viewer on the lower-index overview window that ends moves to the other overview window, never the shell."
- **Problem:** The bullet doesn't say how the window ends. When the viewed lower-index overview window ends because its overview strand is replaced, the viewer lands on the replacement window, not the other overview window. That contradicts the D1 replace criterion and the Constraint "a replacement that ends the viewed overview window selects its new window".
- **Evidence:** `ReplaceStrand` calls `killStrandPanes(st, paneIDs, !replacingWithOverview)` (`internal/reedengine/replace.go:71`). That call passes `selectOnEnd=false`, so the `selectRelevantWindowLocked` call (`internal/reedengine/strand.go:855-856`) is skipped. `selectStrandWindowLocked(strand.PaneID)` then selects the new window (`replace.go:97-98`).
- **Suggested fix:** limit the bullet to a window that ends by removal or by its pane exiting, which `RemoveStrand` and reconcile handle, and keep replacement under the D1 criterion.

F1 (D5 Bound, old-build driver): I found no case where the proposal goes wrong. If the old placement hasn't healed yet, the old `ReplaceStrand` of the runs strand succeeds rather than dropping the strand. "Can drop" still covers that case.

F3 (support-log Q5): no case found.

F4 (fallback while the status strand opened first): no case found. A status strand from the new build has key 1 and never ties. An old-key overview status strand gets replaced at key 1 by the next loom start.

Tool calls:
- grep of `killStrandPanes` in `internal/reedengine`: my context had `replace.go`'s call to it but not its body, which decides whether the ending path selects the relevant window.
