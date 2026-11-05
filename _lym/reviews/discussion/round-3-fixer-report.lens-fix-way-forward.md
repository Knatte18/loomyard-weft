```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal and next step the fix introduced can be carried out and leads to progress.

**MEDIUM: a refused note promotion names a slug that `lym board promote` cannot take.**
- Changed text: D10's added sentence ("also refuses a slug the store holds … naming a different slug as the way forward") and the new D10 acceptance bullet ("a note promotion under a slug the raddle store holds [is] refused with a different slug named").
- Problem: `Board.Promote(idOrSlug any)` (`internal/boardengine/board.go:393`, `store.go:759`) takes only the entry's own slug. A refusal saying "use the slug X instead" (`newSlugError`, `slugrule.go:54`) therefore names a step that promote cannot carry out.
- What works instead: rename the note first, then promote it. `checkNewSlugs` judges only task and project kinds (`slugrule.go:26`), so renaming a note is never refused.
- The fix states neither that sequence nor a promotion-specific way forward. That includes a Sketch whose slug a stored proposal filed: D2 names `lym board rename` for batten's guard, but the board's promote refusal does not.
- Suggested change: for a refused promotion, have D10 and its bullet name `lym board rename &lt;slug&gt; &lt;new&gt;` and then the promote.

**LOW: the offered slug can itself be a store slug.**
- Changed text: the same D10 sentence.
- Problem: `newSlugError` checks its suggestion only through `validate`, the pair seam (`slugrule.go:49`). The fix's new Fact says that seam reads no raddle store, and D10 does not say the store refusal joins the seam. So the suggested `&lt;slug&gt;-task` or warp slug can be one the store holds, and the next write is refused again.
- Suggested change: have D10 say the suggestion is checked against the store too, or that the store check runs inside `validate`.

The fix's other ways forward hold:
- D9 step 5's skip-and-marker rule can be reached on a retry.
- D11's order of mail and marker is consistent with the re-invoke policy.
- D12's fall back to `SessionUsage` exists for every engine.

Tool calls:
- I grepped `boardengine` and `boardcli` for the promote signature and note-rename handling. My context did not show whether promote accepts a new slug.
