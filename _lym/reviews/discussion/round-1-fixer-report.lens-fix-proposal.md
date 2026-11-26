```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (fix-proposal lens).

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F10, F11, F12, F13. F9 was fixed differently and is out of this lens.

1. **MEDIUM: F2's copy can disagree with the landing notice when the setting flips on the last tick before Land.**
   - Changed text: D3, "While a run is ahead of its Land row, the watch copies the land_notice reading the landing notice holds for it into its own memory (D4), once that reading exists." D1 places the watch only "among the run steps, before the done-run clear".
   - Case: on the last tick before Land, the watch runs before `landNotices`, so it copies the previous tick's reading. `landNotices` then reads land_notice afresh and stores the new value (`internal/dobbyengine/landnotice.go:124-126`). On the next tick the run is past Land, and the landing notice decides on the new value (`landnotice.go:142-149`), while the watch's copy can no longer refresh. If land_notice flipped from off to on, the orch gets both lines. If it flipped from on to off, it gets neither and never hears of the landing.
   - Evidence: the old rationale ("the two never disagree") was dropped, and nothing replaces it.
   - Fix: order the watch after `landNotices` in the run steps (`internal/dobbyengine/loop.go:719`). Its last copy before Land is then the reading the landing notice decides with.

2. **LOW: F1's drop of an unlisted run's watch entry loses the memory when a status read fails for one tick.**
   - Changed text: D4, "only the watch, for a run no longer listed that owes no line, and the done-run clear drop a run's entry."
   - Case: `dobbyParentRuns` leaves out a run whose status is unreadable, with a Warn (`internal/battencli/dobbyreads.go:104-107`). That tick the watch drops the run's entry. The next tick it treats the run as first seen and posts the current stop line again, against D4's "repeats no line".
   - Worse, after Worktree-Teardown the copied land_notice reading is gone and cannot be copied again, because the landing notice's memory has already been dropped. A land_notice-on run then posts a landed watch line beside its landing notice, against the acceptance bullet "none when it is on".
   - Fix: drop the entry only once the run is gone from batten's records, not on one missed listing. Alternatively, keep the copied reading once the run is past Land until the done-run clear drops it.

No further tool calls made.
