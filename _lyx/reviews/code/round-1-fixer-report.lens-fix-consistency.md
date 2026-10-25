```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for consistency with its surroundings (lens fix-consistency).

The counts, cross-references and sibling files agree: the summary-table rows sum to each setting's findings, the Limits bullet matches the one unchecked row, the F8 count of 4 matches the list, and `claude-reference.sh` reads columns 1 to 5 of `match.tsv`, so the new `claude_severity` column does not break it. I found two stale statements.

1. **LOW** — `report.md`, `### Codex runs`: "`CODEX_HOME` ... set up by `data/harness/setup-codex-home.sh`". The Data bullet for `harness/` says the same of `usage.sh` and the `usage.txt` it "produced".
   - **Problem:** the scratch `CODEX_HOME` and its `auth.json` link already existed at 11:42, before the bench, and the original `usage.txt` came from the scratch `.scratch/bench/usage.txt`. Both scripts were written afterwards to reconstruct that work.
   - **Evidence:** `ls` showed the link created 2026-10-25 11:42, and `usage.sh` was written during this fix round. The review's F11 `fix` offered "say ... that they were done by hand" as the alternative.
   - **Wording to change:** say the link and the first `usage.txt` were made by hand and the scripts reproduce them.

2. **LOW** — `report.md`, `### Grading`: "The fixer report and bouncer verdict of the rounds holding `codex-only` rows (`D1`, `D2`, `D3`, `D4`, `D6`, `P1`, `P3`, `P4`) were searched for each finding's topic."
   - **Problem:** the fixer's context shows no fixer-report search for `P4` (only the plan files and decision record were grepped for `Takeover`), and the `D3` and `P3` searches were narrow (`pattern`, and a read of the `F1` and `F3` lines).
   - **Evidence:** the weft-side commands in context for `P4` read only `planfiles.txt` and `_lyx/plan` and `_lyx/discussion` greps.
   - **Wording to change:** either run the `P4` search or drop `P4` from the list and describe the other searches as partial.

Reads I made: none; I made no tool calls.
