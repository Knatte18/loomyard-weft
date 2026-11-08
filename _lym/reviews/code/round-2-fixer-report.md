# Fixer-report, code review round 2

## Changes

- F1 (LOW), fixed differently from the proposal, in `_lym/landing/summary.md`.
  - Numbers section: dropped is now 188 and kept 64, not the review's 187 and 65.
    The review's 65 counts the `+` lines of `f2ac2bf7..HEAD`, which includes the reworded `TestCwdMutation_MigratedFilesStayChdirFree` `proves` line.
    That line is not a restored test: `git diff feaa74d0 HEAD` shows it as a `-`/`+` pair next to ten genuinely new keep lines, so 54 + 10 = 64 and 198 − 10 = 188.
  - Added a "Restored after the first review" list of those ten tests with their keep reasons.
  - Relabelled the timing table's second column as measured before the ten restored tests, because its figures predate them.
  - Dropped the stale "14" from the deleted-files bullet: one test file came back, so the count is now 13.
  - No commit: `_lym` is owned by the loop and the file is untracked, so the change is in the working tree only.

## Self-check

- fix-code-correctness, LOW: timing table stale after the restored tests. Fixed with the relabel above.
- fix-code-correctness, LOW: 64 versus the review's 65. Recorded above as fixed differently; no further change.
- fix-proposal, LOW: timing table stale. Duplicate of the item above; fixed.
- fix-proposal, LOW: dropped-cost sentence removed rather than recomputed. Fixed by stating that the 51.5 s total covered the original 198.
- fix-consistency, MEDIUM: "14 files" tally wrong. Fixed by removing the number.
- fix-consistency, MEDIUM: timing table stale. Duplicate; fixed.
- fix-consistency, LOW: dropped-cost wording. Duplicate of the fix-proposal item; fixed.
- fix-consistency, LOW: 64 versus 65 unexplained. Explained in this report; the summary needs no note, as the review's count is not part of it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

The tier 1 gate was not rerun, so the table holds no figures for the final tree.
Rerunning it to refresh the figures is outside this finding.

## Rulings

None.
