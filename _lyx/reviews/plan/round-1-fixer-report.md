# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 1's ordered `--work` checks drop the `--work contains --out` step and say the overlap check with `--out`'s directory refuses a `--work` holding `--out`; the `TestForkauditRefusals` row with `--out` inside a subdirectory of `--work` now expects the overlap reason naming `--out`'s directory.
- **F2** (fixed as proposed): card 5's `--code` refusal row passes `--records` naming an existing empty `t.TempDir()`, and card 5's Intent fixes the order, `--code` before `--records`.
- **F3** (fixed as proposed): card 1's Intent limits the `--out` directory-missing refusal to an explicit `--out`; a defaulted `--out`'s `.scratch/` is created with `mkdir -p` after every check has passed, beside `--work`'s creation, and the header comment says so.
- **F4** (fixed as proposed): card 1's single-spelling rule now covers `forkaudit.sh` and the `.jq` files only, with the Go tests building paths from `internal/lyxdirs`'s `LyxDirName` and `DotLyxDirName`.
  Card 4, the first card whose tests build those paths, lists both constants under `Uses:`.
- **F5** (fixed as proposed): card 4's Intent names the resolved `--records` path in the header on every run; card 5's Intent names the resolved `--code` path under `--pack-check`.
- **F6** (fixed as proposed): card 2's Intent has the marker-table comment and the script's header comment state that a task adding a Webster pack layout adds its marker row in the same task.

## Disputed

None.

## Deferred

None.
