# Fixer report — code round 1

All twelve findings are fixed.
Nothing was disputed or deferred.
Commits are one per finding group, not one per finding, where one script or one report section carried several findings; the commit messages name each finding.
No test suite applies: the changes are documentation, data files and shell scripts under `docs/research/codex-review-bench-2026-10-25/`.
`grid.sh` was syntax-checked and its window reader run on a bench rollout, but `grid.sh` was not run as a whole, since a run spends plan limits.
`usage.sh` reproduces the old columns of `usage.txt` exactly.

## Changes

- F1 (fixed as proposed): `data/usage.txt` gains `five_hour_used_percent` and `weekly_used_percent` per run, read from each run's rollout by the new `harness/usage.sh` and `harness/windows.sh`.
  The report's Data bullet lists the columns.
  The per-run points in the Billing section (Sol 2 to 5 except `P1` 10 and `P4` 7, Astra 5 to 17, the 68 and 79 percent readings) match the readings; nothing needed correcting.
- F2 (fixed as proposed, with extra evidence): `Codex runs` states that the byte-identical criterion is met only as far as the file time shows, records the hash read after the bench, and states the `auth.json` symlink facts.
  A second read at 12:48 showed `config.toml` rewritten by a live operator `codex` session that started before the bench; the report and the new `data/operator-codex-home.txt` record both hashes and the `ls -A` listing.
  The 12:15 directory change stays unexplained.
  The conclusion carries a one-line statement.
- F3 (fixed as proposed): 11 of the 12 `codex-only unchecked` rows are graded by reading the artifact lines and code at the input's code commit: 8 `codex-only real`, 3 `false`, and `D4` Luna F3 stays unchecked with its reason in the `note` column.
  Fixer reports and bouncer verdicts of the rounds were searched by keyword; none touches a codex-only finding except as noted in the `D2` Luna F1 row.
  The summary table, Conclusion, Grading section, findings list and Limits follow the new counts.
- F4 (fixed as proposed): `harness/grid.sh` reads both windows after each run, writes them to `result.txt` and exits at 70 percent or more.
  The report keeps its statement that the stop did not fire during the bench, and states that the check runs after a run, so a rerun started above 70 spends its first cell and refused runs do not stop the grid.
- F5 (fixed as proposed): `data/match.tsv` gains `claude_severity`; new `data/claude-findings.tsv` and `data/claude-usage.tsv`, produced by `harness/claude-reference.sh`.
  The 134 total, 21 `BLOCKING`, the per-setting found counts and the Claude tokens and wall times recompute from them.
- F6 (fixed as proposed): `Codex runs` states that `workspace-write` confines writes and not reads, gives the `grep` over all 31 `events.jsonl` that matches no `loomyard-weft` or `bench/inputs` path, and names the one run that searched `/home/knatte/Code` for `enclose.go` files.
  The prompt section no longer implies confined reads.
- F7 (fixed as proposed): `grid.sh` skips a cell only when its `result.txt` has `rc=0`.
- F8 (fixed with own change): the conclusion now says 4 of Sol's 11 held-up findings were checked against the code, not 2 of 6.
  The proposed 2 was right for the 6 findings graded when the review was written; the F3 grading added `P1` F3 and `P3` F3 as code-checked, so 2 would be stale.
- F9 (fixed as proposed): `fill()` in `harness/build.sh` quotes the replacement, and the closing check exits 1 on a leftover `{{`.
  Checked with `Q&R`, which now stays literal.
- F10 (fixed as proposed): `grid.sh` picks the rollout newer than a marker file touched before the run and reads nothing when none is.
- F11 (fixed as proposed, report wording differs): `grid.sh` calls the committed `run.sh` through its own directory.
  `harness/setup-codex-home.sh` and `harness/usage.sh` are new.
  The Data section says the original link and `usage.txt` were made by hand and the scripts reproduce them.
- F12 (fixed as proposed): `report.md` is broken at independent-clause boundaries in the lines the review listed and the same pattern elsewhere, list continuations indented and table rows left alone.

## Self-check

- fix-code-correctness, MEDIUM: `usage.sh` took the oldest matching rollout, which after a rerun is the refused run's. Fixed: `tail -1`; output still reproduces `usage.txt`.
- fix-consistency, LOW: the report said the `auth.json` link and `usage.txt` came from the scripts. Fixed: the report says they were made by hand and the scripts reproduce them.
- fix-consistency, LOW: the Grading section overstated the fixer-report and bouncer-verdict reading. Fixed: described as keyword searches, not line-by-line reads.
- fix-affected-set, LOW: four lines still joined clauses with `, so`. Fixed.
- fix-proposal, LOW: `grid.sh` checks the stop only after a run, so a rerun above 70 spends one cell. Fixed with documentation in the report, not code: a pre-run check against the newest rollout would block a rerun after a window reset on a stale reading.
- fix-proposal, LOW: refused runs write no reading, so they do not trigger the stop. Fixed with the same report sentence.
- fix-proposal, LOW: `setup-codex-home.sh` replaces an existing `auth.json`. Fixed: the script header says so.

## Disputed

None.

## Deferred

None.
