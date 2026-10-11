# Tokencount default counts only full loom runs

`tokencount-full-runs` · Task · enhancement, shed · running · Worktree-Create

## Seen (operator, 2026-10-11)

`tools/tokencount` with no slugs counts the most recently active runs (`-last`, default 6).
On 2026-10-11 that picked only the Discussion bench pairs (`twb-combo-*`, `twb-twc-*`, `twb-twp-*`), which ran Discussion-Write and stopped, so `.scratch/token-usage-by-role.md` showed a Discussion-only role mix and no Plan, Webster or review rows.

## Wanted

- The default selection counts only full loom runs: a run whose loom lifecycle went through to the end (landed, or reached Finalize), never a run stopped partway such as a bench pair or an abandoned run.
- `-last N` means the N most recent full loom runs.
- Slugs named on the command line are still counted as given, so a partial run can be inspected on purpose.
- The report header says which runs were counted and, when the default skipped recent partial runs, says so in one line.
- Update the package's doc comment and `tokencount.sh` / `tokencount.cmd` comments to match.
