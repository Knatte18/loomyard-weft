```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling text it touches (the fix-consistency lens).

Changed text checked: the D2 viewer paragraph, the D3 Title and collision paragraphs, the D3 rationale, the Constraints bullet, the deleted D2 viewer criterion, the D3 and D4 criteria, and the support-log rejected alternative.

1. **MEDIUM.** D2's new sentence "The runs strand's new overview window is never selected" contradicts D2's own failure bullet.
   That bullet says that after a failed move, "a `loom-status` strand still present is replaced by the runs strand, as today".
   Today's replace with an overview spec selects the new overview window when it ended the viewed window: `endedCurrent && replacingWithOverview` → `selectStrandWindowLocked`, `internal/reedengine/replace.go:97-98`.
   So on that fallback path the overview window is selected.

2. **MEDIUM.** D2's "with none live, nowhere, leaving the choice to tmux" disagrees with D2 case 2, where a runs strand already exists.
   In case 2, replacing a dead `loom-status` in its own window ends that window.
   `relevantWindow` then falls back to the overview strand's window when no agent window is live (`internal/reedengine/ownwindow.go:467-469`), so a viewer there lands on the beam overview, not nowhere.
   The paragraph sits under D2 as a whole, so it covers case 2 as well as case 1.
   "Nowhere" holds only in case 1, where no overview strand exists at the moment of the replace.

3. **LOW.** The support log was left stale beside the D2 change.
   `_lym/discussion/support-log.md:20` still reads "bound-breaker 5 and existing-paths 3 (viewer landing unstated): fixed, D2 states the viewer ends on the beam overview, with a criterion."
   D2 no longer says that, and the criterion is deleted.
   The twin of this fix in the support log was not updated.

Further tool calls: none.
