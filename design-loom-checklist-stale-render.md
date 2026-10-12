# Loom status pane keeps an old render under the new one

`loom-checklist-stale-render` · Note · bug, loom

The operator saw two loom checklist renders stacked in one run window's status pane on 2026-10-12 (task `run-robustness`): an older `loom paused | now Webster-Burler-Review` block, with a right-hand column left over from a wider layout, above the current `loom running | now Merge-In` block.

The old render is not cleared when the new one is drawn, so the pane mixes two states and reads as if boxes were assigned at random.

## What to do

- Clear or overwrite the whole pane on every redraw, including after a resize or a layout change from two columns to one.
- Pin it with a test that redraws a shorter, narrower checklist over a longer, wider one.
