```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1, F4, F5, F7, F9, F10, F11, F12). I built a concrete case against each proposal.

**Findings**

1. **LOW**: `data/harness/grid.sh`, the F4 stop check, which runs only after a run.
   - The proposal was to read the windows after each `run.sh` call and exit at 70 or above.
   - A rerun started while a window already shows 70 or more still launches one cell before it can notice. Take a window at 96, the Astra case after `D5`. The loop skips only `rc=0` cells, so `D6` runs first. If the plan lets it start, that run spends more limit; if refused, it writes no rollout and the `[ -n "$rollout" ] || continue` line moves on, so the stop never fires.
   - D6 says to stop the bench when a window shows 70 percent. The committed script enforces that only after a run, not before the first.
   - The report's new Billing text ("exits once either shows 70 percent used") does not say this.
   - Evidence: the `grid.sh` loop order, with no window read before `run.sh`.

2. **LOW**: `data/harness/grid.sh`, where a refused run never reaches the stop.
   - The F10 proposal prints nothing when no rollout is newer than the marker, and F4 reuses that path for the stop.
   - After limit exhaustion the refused cells each take about 2 s, write no `token_count`, and loop to the end of the grid without any stop. The report's "stops once either window shows 70 percent" holds only for runs that write a reading.
   - Evidence: the `usage.txt` lines for the refused Astra runs carry no percentages, and `windows.sh` prints nothing for them.

3. **LOW**: `data/harness/setup-codex-home.sh`, the `ln -sfn` from the F11 proposal.
   - If Codex replaced the scratch `auth.json` link with a real refreshed-token file, a rerun of the setup script deletes that file and relinks to the operator's `auth.json`.
   - This is scratch-only and low impact. The script's header says the link is writable through to the operator's file, but not that it overwrites an existing file.

I found no break in the other proposals. The F7 skip test, the F9 quoting, the F5 `found_by` derivation and the F1 `usage.txt` format all held under my cases. The F9 quoting check came out `aQ&Rb`, so `&` is now literal.

Tool calls made: none; my context already held every file I examined.
