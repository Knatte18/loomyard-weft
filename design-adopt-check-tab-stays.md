# The adopt-check tab stays open and takes space after the check ends

`adopt-check-tab-stays` · Note · bug, reed

## Seen (operator, 2026-11-11)

After `lym adopt root` ran its check on OSL-1046, an `adopt-check` tab stayed open at the bottom of the terminal and took a lot of space.
The check had ended green in all three tiers, and the tab was still there.

## Idea

Close the check's pane or tab when the adopt check ends, or shrink it to one line, since its result is already in the adopt record and the creel mail.
Read first which part of `lym adopt` opens the tab, whether a failed check should keep it open for the operator, and where the suite log stays readable after the close.
