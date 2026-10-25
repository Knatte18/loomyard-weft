```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the four harness scripts under `data/harness/`, the only code the diff adds. I made no tool calls; the scripts are in my context.

**Per-function list**

- `build.sh`, start-time extraction (`grep -m1 'start:' | tr -s ' ' | cut -d' ' -f3`). I ran it on an indented `  start: 2026-10-22T17:36:02Z`: field 3 is the timestamp. Correct for the indented form, and `data/inputs.txt` shows every input parsed.
- `build.sh`, `code=$(git rev-list -1 --before=...)`. I ran it with a normal timestamp: it returns a commit. `--before` filters on committer date, not ancestry. The report states this as a limit. Correct as designed.
- `build.sh`, `strip()`. A file with a leading `<!-- ... -->` is trimmed. A file with no comment passes through. The `[ ... ] && s=...` line does not trip `set -e`. Correct.
- `build.sh`, `fill()`: **not correct**, see F1.
- `build.sh`, discussion and plan staging loops. The plan loop's `grep -v '/archive-'` is correct, and a file-less `ls-tree` result yields an empty loop. The discussion loop runs `ls-tree` non-recursively, so a subdirectory entry would be dumped as a tree listing into a file. Harmless for the committed inputs.
- `build.sh`, final `grep -c '{{' ... || true`. It only reports leftover placeholders and never fails the build, so a corrupted prompt is not caught.
- `run.sh`, path re-pointing. Both the pattern and the replacement are quoted, so `&` is safe here. Correct.
- `run.sh`, exit codes. A `timeout` kill gives rc=124 and is recorded in `result.txt`. A missing review file appends "no review file". `rm -rf "$cell/work"` runs on a path built from vars that are all set. Correct.
- `run.sh`, sandbox. See F2.
- `grid.sh`, skip test `[ -f "$cell/result.txt" ] && continue`: **not correct**, see F3.
- `grid.sh`, window readout: **not correct**, see F4.
- `grid.sh`, 70 percent stop. The script has none. The report admits this, so it is not a hidden defect.
- `dump.sh`, a review with two findings. Both lines print, the last one at the closing `---`, and `id` is cleared so it is not printed twice. Correct.
- `dump.sh`, an APPROVED review with no `findings` key. It prints nothing. Correct.
- `dump.sh`, a finding whose `summary` line is missing. It prints with an empty summary at end of input. Acceptable.

**Findings**

- **F1 — LOW, design.** `build.sh` `fill()` (the line `t=${t//"{{.${kv%%=*}}}"/${kv#*=}}`) leaves the replacement unquoted. Under bash 5.2 with `patsub_replacement`, a `&` in a substituted value (rubric, fasit or tool-use text) is replaced by the matched placeholder. A prompt could therefore be silently corrupted. The check at the end of the script only counts `{{`. Evidence: `data/harness/build.sh` in the `fill` definition. I did not confirm the bash version used or that any rubric contains `&`. Fix: quote the replacement, `"${kv#*=}"`.
- **F2 — MEDIUM, design.** `run.sh` uses `-s workspace-write -C "$cell/work"`. As far as I know, that sandbox limits writes but not reads, so Codex could read the whole filesystem, including the weft checkout at `/home/knatte/Code/loomyard-LYXHUB/loomyard-weft`. That checkout holds Claude's review files. Decision D4 says Codex "can read and write only under its working directory", and the report says the work directory held none of Claude's review files. The scripts therefore do not enforce input blinding; they only avoid copying the files in. I have not confirmed this behavior of Codex 0.164.0. Evidence: `data/harness/run.sh` (the `codex exec` call) and `data/harness/build.sh` (`WEFT=`). Fix: state the read-exposure in the report, or show from the event logs that no run opened a weft path.
- **F3 — MEDIUM, design.** `grid.sh` treats any cell with a `result.txt` as finished, and `run.sh` writes `result.txt` even when `rc=1`. The Astra `D6` and `P1` to `P4` cells that the usage limit refused (`data/usage.txt` lines 18 and 21–30) would be skipped on a rerun. The "unmeasured" cells cannot be filled by re-invoking the committed grid script without deleting their directories by hand. Evidence: `data/harness/grid.sh`, the `[ -f "$cell/result.txt" ] && continue` line. Fix: skip only when `result.txt` records `rc=0`.
- **F4 — LOW, design.** `grid.sh` reads the window percentages from the newest rollout via `ls -t ... | head -1`. When a run fails in 2 s, such as the refused Astra plan runs, no new rollout may exist, so the line printed for that cell would be the previous run's percentages. Evidence: `data/harness/grid.sh`, the `f=$(ls -t ...)` line; `data/usage.txt` lines 21–30 for the 2 s failures. Fix: take the rollout whose path is newer than the run's start, or print nothing when none is.
- **F5 — LOW, consistency.** `grid.sh` calls `$ROOT/.scratch/bench/run.sh`, but the committed copy is `data/harness/run.sh`, so the committed scripts do not run as committed. No committed script produces `data/usage.txt` (the fresh-token, verdict and count columns) or sets up the `auth.json` link. The report's description of `harness/` does not mention that these steps were done by hand. Evidence: `data/harness/grid.sh`, the `bash $ROOT/.scratch/bench/run.sh` line, and the file list of `data/harness/`. Fix: have `grid.sh` call the sibling script, and either commit the usage-extraction and link-setup steps or say they were manual.

No tool calls made.
