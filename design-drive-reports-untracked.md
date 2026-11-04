# loom drive writes untracked files under _lym/; _lym is expected to be fully tracked

`drive-reports-untracked` · Task · bug, loom, shed, fabric · running · Worktree-Create

Imported from [issue #43](https://github.com/Knatte18/loomyard/issues/43).

## Observed

In a hub's weft checkout, `git status` shows 1580 untracked files, all under `<project>/_lym/shed/<slug>/`:

- 1563 `drive-reports/drive-envelope-<timestamp>.json` (one slug)
- a few `drive-reports/` directories for other slugs
- `tackler/` and `arbiter/` directories
- 1 modified file: `<slug>/status.json`

Timestamps of the first three drive envelopes: 08:38:00, 08:38:26 and 08:38:56, so a new file roughly every 30 seconds while `lym loom drive` runs.
The newest was written the next morning at 09:06.
Later weft sync commits (20:44 and 21:19 the same day) did not include any of them.
There is no `.gitignore` entry for them, and 534 other files under `_lym/` are tracked.

## Expected

Everything under `_lym/` is tracked in git.
Files that must not be tracked live under `.lym/`.

## Impact

- The files are neither synced nor ignored, so `git status` is permanently dirty and the work would be lost if the checkout is lost (found while preparing an OS upgrade).
- If `drive-reports` is meant to be transient, it belongs in `.lym/`.
- The rate (one file per ~30 s) may itself be a bug; please check whether a report is written per poll rather than per drive.

## Environment

loomyard built from main, Ubuntu 24.04, `lym loom drive` and `lym orch watch` running.
