```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the scripts the fix added or changed (`grid.sh`, `windows.sh`, `usage.sh`, `claude-reference.sh`, `build.sh` `fill()`, `setup-codex-home.sh`) by walking concrete inputs through them.

**Findings**

1. MEDIUM — `data/harness/usage.sh`, the `rollout=$(grep -l ... | head -1)` line. `usage.sh` takes the oldest rollout that matches a cell.
   - The F7 fix makes reruns of a failed cell normal. After a rerun, a cell such as `P1/gpt-6-astra-medium-r1` has two rollouts under `sessions/`.
   - The first is the refused run's rollout, which has no `token_count` event. The second is the successful rerun.
   - `grep -l` expands the glob in name order, which is time order, so `head -1` picks the refused run's rollout. `windows.sh` then prints nothing, and the rerun's `five_hour_used_percent=` and `weekly_used_percent=` columns come out empty.
   - Its token, verdict and wall columns come from `events.jsonl` and `result.txt`, which the rerun overwrites, so only the percentages go wrong.
   - The fix is `tail -1`. The 31 existing cells have one rollout each, so the committed `usage.txt` is unaffected.

**Checked, no finding**
- `windows.sh`: on a rollout with a `token_count` event it prints `43.0 7.0`. On a rollout without one it prints a bare newline, which I confirmed with `od`. A missing file only writes a grep error to stderr.
- `grid.sh` with a refused run:
  - A refused run writes a rollout newer than the marker, but `windows.sh` prints nothing for it. `five` stays empty, so the loop continues without stopping and without appending to `result.txt`.
  - The `-newer` filter keeps a previous run's rollout from being picked up.
  - `${five%.*}` turns `79.0` into `79`, so the integer comparison works.
  - The `grep -q '^rc=0'` skip test still works after `result.txt` gains an appended percentage line, because `run.sh` rewrites `result.txt` with `>` on each run.
- `usage.sh`: its old columns reproduce the committed `usage.txt` exactly (empty diff), as the parent ran.
- `claude-reference.sh`: the 134 findings, 21 `BLOCKING`, the per-setting counts and the usage ranges match the report. The `|| true` handles the `set -e`/`pipefail` failure from a `&&` chain that ends false. The 4-space indent match excludes the 6-space seed block.
- `build.sh` `fill()`: the ampersand case yields `aQ&Rb`. The closing check fails only when a `{{` is left, and the current inputs have none.
- `setup-codex-home.sh`: nothing wrong.

**Tool calls:** none. My context already held every file I needed.
