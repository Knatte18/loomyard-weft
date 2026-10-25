```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran each set-shaped finding's own search against the changed tree.

**Findings**

1. LOW — F12, `report.md` Limits section, the `- One run per cell: no variance is measured, so Sol and Astra, which tie on the five shared inputs, cannot be ranked.` bullet (line 235).
   - The review named it (original line 159) as a line to break at its clause boundary, and it is still one line.
   - Evidence: the F12 search still returns line 235 with the `, so` clause unbroken.
   - Two similar lines also remain whole: line 22 (`One grader read each finding, so these rates are not validated.`) and line 148 (`... , so that evidence confirms or refutes none of the labels.`).
   - Line 225 (`It also skips only cells that finished with rc=0, so the refused Astra cells run again on a rerun.`) is the same shape.
   - All four were written or kept by the fix and join two clauses with `, so`.

**Sets that hold**
- F3: `match.tsv` has exactly one `codex-only unchecked` row (D4 luna F3), and its note gives the reason.
- F3 counts: the table, Conclusion and Limits agree with `match.tsv`. Luna is 2 real, 1 unchecked, 3 false; Sol is 11 real; Astra is 5 real.
- F3: no stale "12 of 68", "12 Codex" or "6 findings" text remains.
- F5: no `match.tsv` row with a Claude finding lacks a `claude_severity`. All 46 shared rows are covered.
- F1: no `usage.txt` line lacks the window columns. Refused runs carry empty values.
- F11: `grid.sh` calls the committed `run.sh` through `$here`, and no committed script refers to a `.scratch/bench/*.sh` copy.

**Further calls**

Only the one Bash call above, a combined rerun of the F12, F3, F5, F1 and F11 searches. The context did not hold the post-edit `report.md` line set or the post-edit `match.tsv` and `usage.txt` counts.
