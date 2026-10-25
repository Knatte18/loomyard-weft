---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: docs/research/codex-review-bench-2026-10-25/data/usage.txt:1
    summary: usage.txt carries no window percentages although D6 requires them per run and the report's billing figures rest on them
    fix: "Add per-run `five_hour_used_percent` and `weekly_used_percent` columns to `data/usage.txt`, read from each run's rollout `token_count` rate_limits, which are still under the scratch `CODEX_HOME/sessions/`. Then correct the Data section of `report.md` (the `usage.txt` bullet) so it lists the new columns, and check the per-run point figures in the Billing section (Sol 2 to 5 each except P1 10 and P4 7, Astra 5 to 17 each, the 68 and 79 percent readings after Astra D3 and D4) against them."
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: docs/research/codex-review-bench-2026-10-25/report.md:86
    summary: the acceptance criterion that ~/.codex/config.toml is byte-identical before and after is not evidenced, and a ~/.codex directory change at 12:15 is left unexplained
    fix: "Record now, in `report.md` under `### Codex runs`, the `sha256sum` and mtime of `~/.codex/config.toml`, and the entry names of `~/.codex` (`ls -A`) with their mtimes, and say which entry the 12:15 change belongs to or that it is still unexplained. Change the conclusion-level wording so the report states plainly that the byte-identical criterion is met only as far as the file mtime shows, not as hashed. Also say in the same section that `.scratch/codex-home/auth.json` is a writable symlink to `~/.codex/auth.json`, and that the file's mtime (2026-10-24 11:16) shows no token refresh wrote through it."
    origin: handler
  - id: F3
    severity: MEDIUM
    class: scope
    location: docs/research/codex-review-bench-2026-10-25/data/match.tsv:1
    summary: D5 asks every codex-only finding to be judged against the artifact and code and the fixer report and bouncer verdict read as evidence, but 12 of 68 findings stay unchecked and no fixer or bouncer evidence was read
    fix: "Grade each of the 12 `codex-only unchecked` rows of `data/match.tsv` by reading its cited artifact lines and code at the input's code commit, and relabel each `codex-only real` or `false`. Read each round's fixer report and bouncer verdict from the weft commit for the codex-only rows. Then update the summary table columns, the Conclusion bullet that says no false-finding rate is established, and the Grading section of `report.md` to the new counts. If a row cannot be graded, keep it unchecked and state the reason in its `note` column."
    origin: handler
  - id: F4
    severity: MEDIUM
    class: scope
    location: docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:1
    summary: D6 requires the bench to stop when either plan window shows 70 percent used, but the committed grid script has no check and the run went on to 96 percent
    fix: "In `data/harness/grid.sh`, after each `run.sh` call read the newest rollout's `used_percent` for both windows and `exit` with a message when either is at or above 70, and write the percentages into the cell's `result.txt`. Keep the report's existing statement that the stop did not fire during the bench, and add that the committed `grid.sh` now enforces D6 for any rerun."
    origin: handler
  - id: F5
    severity: MEDIUM
    class: design
    location: docs/research/codex-review-bench-2026-10-25/data/match.tsv:1
    summary: the BLOCKING counts in the table and conclusion cannot be recomputed from the committed data because match.tsv has no Claude severity column and the Claude reference figures are not committed
    fix: "Add a `claude_severity` column to `data/match.tsv` for every matched row, and add a `data/claude-findings.tsv` listing every Claude finding of the ten inputs with id, severity and whether any Codex setting matched it, so the 21 `BLOCKING` total, the 134 total and the claude-only set follow from files. Add the Claude reference tokens and wall times (236k to 489k, 383 s to 735 s) as columns of `data/inputs.txt` or a sibling file, taken from the weft round usage files."
    origin: handler
  - id: F6
    severity: MEDIUM
    class: design
    location: docs/research/codex-review-bench-2026-10-25/data/harness/run.sh:20
    summary: workspace-write limits writes, not reads, so Codex could read Claude's review files in the weft checkout, and D4 and the report claim it could read only its working directory
    fix: "In `report.md` under `### Codex runs`, state that `-s workspace-write` confines writes to the work directory but that reads were not confined, and that input blinding rests on the copied work directory holding no Claude review files. Then show from the committed or scratch `events.jsonl` that no run opened a path under the weft checkout, with the command used, or record that this was not checked. Correct the D4-derived sentence in the report that implies reads were confined."
    origin: lens:code-correctness
  - id: F7
    severity: MEDIUM
    class: design
    location: docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:9
    summary: grid.sh skips any cell with a result.txt, and run.sh writes one on a refused run, so the unmeasured Astra cells cannot be filled by rerunning the committed script
    fix: "In `data/harness/grid.sh` change the skip test to `grep -q '^rc=0' \"$cell/result.txt\" 2>/dev/null && continue`, so a cell whose run failed is run again."
    origin: lens:code-correctness
  - id: F8
    severity: LOW
    class: consistency
    location: docs/research/codex-review-bench-2026-10-25/report.md:13
    summary: the conclusion says one of Sol's six codex-only real findings is against the code, but the report's own list checks two against code, D4 and P1
    fix: "In `report.md` line 13 change `1 of them against the code` to `2 of them against the code`, matching the `D4` bullet checked against `internal/tenderengine/processes.go` and the `P1` bullet checked against the snapshot in the section `What Codex found that Claude's round did not`."
    origin: handler
  - id: F9
    severity: LOW
    class: design
    location: docs/research/codex-review-bench-2026-10-25/data/harness/build.sh:34
    summary: fill() leaves the replacement unquoted, so under bash patsub_replacement an ampersand in a substituted value is replaced by the matched placeholder and silently corrupts the prompt
    fix: "In `fill()` in `data/harness/build.sh` quote the replacement, `t=${t//\"{{.${kv%%=*}}}\"/\"${kv#*=}\"}`, and make the closing `grep -c '{{'` line fail the script when it finds a leftover placeholder."
    origin: lens:code-correctness
  - id: F10
    severity: LOW
    class: design
    location: docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:11
    summary: the window readout takes the newest rollout file, which for a run refused in 2 s may be the previous run's, so the printed percentages for refused cells can be stale
    fix: "In `data/harness/grid.sh` select the rollout with `find ... -newer \"$cell/result.txt\"` or by a start-time marker file, and print nothing when no rollout is newer than the run's start."
    origin: lens:code-correctness
  - id: F11
    severity: LOW
    class: consistency
    location: docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:12
    summary: grid.sh calls $ROOT/.scratch/bench/run.sh, so the committed scripts do not run as committed, and no committed script builds usage.txt or links auth.json
    fix: "Make `data/harness/grid.sh` call `\"$(dirname \"$0\")/run.sh\"`, add the usage-extraction and `auth.json` link-setup commands as scripts under `data/harness/` or say in the Data section of `report.md` that they were done by hand, and fix the `harness/` bullet there to match."
    origin: lens:code-correctness
  - id: F12
    severity: LOW
    class: consistency
    location: docs/research/codex-review-bench-2026-10-25/report.md:9
    summary: report.md holds lines with two independent clauses joined by a semicolon or a comma and a conjunction, against the semantic line break rule
    fix: "In `report.md` break each such line at the clause boundary so the second clause starts a new line, at lines 9, 16, 40, 41, 42, 64, 66, 86, 88, 96, 129, 132, 133, 147, 151, 155, 159, 162 and 163, keeping table rows and list-item continuations indented as they are."
    origin: lens:code-pattern-fit
---

### [MEDIUM] usage.txt carries no window percentages

**Location:** `docs/research/codex-review-bench-2026-10-25/data/usage.txt:1`
**Issue:** D6 has the bench record the percent of each plan window used after each run, and the acceptance criteria want what each run cost against the plan's limits.
Each `usage.txt` line carries return code, wall time, tokens, verdict and finding count only.
`grid.sh` prints the window figures to stdout, and the rollout files they come from are under `.scratch/`, so the report's per-run points, its 68 and 79 percent readings and its 0 to 96 percent span cannot be re-read from the committed data.

### [MEDIUM] config.toml byte-identical is not evidenced

**Location:** `docs/research/codex-review-bench-2026-10-25/report.md:86`
**Issue:** The acceptance criteria and the board entry require the operator's `~/.codex` to be untouched, with `config.toml` byte-identical before and after.
The report concedes no hash was taken and rests on a file time of 2026-10-24 16:52.
It also notes `~/.codex` itself changed at 12:15 on 2026-10-25, inside the bench window, with no file explaining it; that directory mtime is still 12:15:56 now.
`.scratch/codex-home/auth.json` is a plain symlink to `~/.codex/auth.json`, not a read-only link as D4 and the focus ask; its target's mtime (2026-10-24 11:16) shows no refresh wrote through it, but nothing in the report says so.
The criterion is honestly disclosed as unproven, yet unproven is not met.

### [MEDIUM] D5 grading left 12 findings unchecked and read no fixer or bouncer evidence

**Location:** `docs/research/codex-review-bench-2026-10-25/data/match.tsv:1`
**Issue:** D5 says `codex-only` findings are judged by reading the cited artifact and code, and the round's fixer report and bouncer verdict are read as evidence of what the pipeline accepted.
`match.tsv` has 12 `codex-only unchecked` rows (5 Luna, 5 Sol, 2 Astra), and the report says the fixer and bouncer evidence was not read.
The `false` column is zero for that reason as much as for Codex's accuracy, and the conclusion's "no false-finding rate is established" is the cost.
The report states both shortfalls, so the finding is that the change the fasit asks for was not made, not that it was hidden.

### [MEDIUM] The 70 percent stop is not implemented in the committed harness

**Location:** `docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:1`
**Issue:** D6 stops the bench when either window shows 70 percent used, and the board entry says to stop early when limits come close to running out.
`grid.sh` has no such check; the report says the window read 68 percent after Astra `D3` and 79 after `D4`, and the grid stopped only when runs were refused after `D5` and `D6`'s 18 s.
The report names this, but the harness it commits still lacks the stop.

### [MEDIUM] BLOCKING counts and the Claude reference cannot be recomputed from the data

**Location:** `docs/research/codex-review-bench-2026-10-25/data/match.tsv:1`
**Issue:** I recounted the table from `match.tsv` and `usage.txt`: findings (18, 35, 15), shared (12, 24, 10), codex-only real (1, 6, 3), unchecked (5, 5, 2), distinct Claude findings found (12, 22, 10; 8, 10, 10 on `D1` to `D5`), fresh input tokens (400k, 862k, 273k), wall times (284, 1936, 628 s), the 134 and 73 totals from the inputs table, and the verdict agreement all follow.
The `x of 21 BLOCKING` figures (1, 7, 2 of 7), "Sol missed all four `BLOCKING` findings of `D4` and all three of `P4`" and the Claude reference tokens and times do not: no committed file records a Claude finding's severity or the Claude round's usage.
D5 makes the match table the audit trail, and the conclusion leans on the 7 of 21.

### [MEDIUM] Reads under workspace-write are not confined to the working directory

**Location:** `docs/research/codex-review-bench-2026-10-25/data/harness/run.sh:20`
**Issue:** `-s workspace-write` confines writes; it does not hide the rest of the filesystem from reads.
D4 says Codex "can read and write only under its working directory", and the report says the work directory held none of Claude's review files, which holds for copies but not for the weft checkout that `build.sh` reads from.
Whether any run read a weft path is not shown, so the blinding that makes the shared and claude-only counts fair rests on an unchecked assumption.

### [MEDIUM] Refused cells are skipped on a rerun

**Location:** `docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:9`
**Issue:** `grid.sh` treats a cell with a `result.txt` as done, and `run.sh` writes one with `rc=1` for the refused Astra `D6` and `P1` to `P4` runs, so rerunning the committed script after the limit resets leaves the unmeasured cells unmeasured.

### [LOW] The conclusion's count of code-checked Sol findings is off by one

**Location:** `docs/research/codex-review-bench-2026-10-25/report.md:13`
**Issue:** The conclusion says Sol added 6 findings that held up, "1 of them against the code".
The report's own list marks `D4` (against `internal/tenderengine/processes.go`) and `P1` (against the snapshot's `internal/shedbuild/build_engines_test.go`) as checked against code, which is 2.

### [LOW] fill() can corrupt the prompt through an ampersand

**Location:** `docs/research/codex-review-bench-2026-10-25/data/harness/build.sh:34`
**Issue:** I ran `t="a{{x}}b"; v="Q&R"; echo "${t//"{{x}}"/${v}}"` under bash 5.3.9, where `patsub_replacement` is on, and got `aQ{{x}}Rb`: the ampersand became the matched placeholder.
None of the era-current rubric and focus stencils I grepped holds an ampersand, so the committed inputs are probably unaffected, and the script's closing check counts `{{` without failing.

### [LOW] Window readout can print the previous run's rollout

**Location:** `docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:11`
**Issue:** `ls -t ... | head -1` picks the newest rollout overall; a run refused in 2 s may write none, so the printed percentages for those cells are the prior run's.

### [LOW] The committed scripts do not run as committed

**Location:** `docs/research/codex-review-bench-2026-10-25/data/harness/grid.sh:12`
**Issue:** `grid.sh` calls `$ROOT/.scratch/bench/run.sh`, and nothing committed builds `usage.txt` or creates the `auth.json` link, so the report's `harness/` description overstates what a reader can rerun.

### [LOW] Semantic line breaks in report.md

**Location:** `docs/research/codex-review-bench-2026-10-25/report.md:9`
**Issue:** `CLAUDE.md` asks for one sentence per line and a break at an internal independent-clause boundary.
The listed lines join two clauses with a semicolon or a comma and a conjunction on one line.
Table cells are on one line, as required.

## Rejected

- Lens code-pattern-fit: the Codex review files under `data/reviews/` break semantic line breaks. They are Codex's verbatim output kept as measurement data, so their wrapping is the record.
- Lens code-tests and lens code-affected-set returned no findings; nothing to judge.

## Notes on the focus questions I settled with no finding

- Diff paths: all 33 paths sit under `docs/research/codex-review-bench-2026-10-25/`; `git ls-files` shows no `.scratch`, snapshot, `CODEX_HOME` or auth file committed.
- Scripts and sandbox: `run.sh` sets `CODEX_HOME` to `.scratch/codex-home`, work directories are `.scratch/bench/runs/<id>/<cell>/work`, never the worktree root, and the call is `-s workspace-write`; `exec` has no approval flag, which the report says. No script writes to `~/.codex`.
- Inputs and commits: the report's inputs table agrees with `data/inputs.txt` on every weft prefix and code commit, and the report says round 1 only.
- Grid coverage: `data/reviews/` matches what the report says ran; the Astra `D6` and `P1` to `P4` cells and the Luna low cells are named unmeasured with the limit as the reason.
- Conclusion strength: the report hedges on one run per cell, the solo versus fan caveat and the single grader, and its "extra, non-gating reviewer" claim is within what the 22 of 134 and 6 real extras support.
