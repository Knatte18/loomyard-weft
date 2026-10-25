# Decision record — Bench Codex Astra against solo Opus as a plan reviewer

## Goal

Measure `gpt-6-astra` at medium effort against Claude `opus` at medium effort as a solo plan reviewer, with no lens forks and the same prompt on the same plan snapshots.
The earlier bench (`docs/research/codex-review-bench-2026-10-25/`) cut Astra off before the plans and compared Codex against an Opus lens fan, so it answered neither like for like.
The deliverable is one report, `docs/research/codex-astra-solo-bench-2026-10-25/report.md`, plus the data it rests on.

## Scope

- Four reviews in all: Astra on `P1` and `P4`, Opus solo on `P1` and `P4`, each once, both at effort medium.
- `P1` is the `gate-runs-once` plan and `P4` the `machine-handover` plan, as the earlier bench's `data/inputs.txt` defines them.
- Inputs, snapshots, reviewer prompt and the `codex exec` line are the earlier bench's, rebuilt with its `data/harness/build.sh` and `run.sh` retargeted at this worktree.
- Opus runs the same `prompt.md` of each input through `lyx shuttle run --model opus --effort medium`.
  The prompt already carries the solo cluster rule, so no forks are spawned.
- The report gives per input the findings each reviewer found, shared, unique and real on reading, unique and false, the verdicts, set against the recorded Claude fan review of the earlier bench; tokens, wall time and the share of the 5-hour window.
- No code under `internal/`, `cmd/` or `contracts/` changes.

## Decisions

### D1: Harness copied into this bench's data directory, paths retargeted

The new bench's `data/harness/` holds copies of `build.sh`, `run.sh`, `setup-codex-home.sh`, `windows.sh` and `dump.sh` with `ROOT` pointing at this worktree.
Rationale: the earlier bench's scripts hard-code its own worktree, and the report directory is the durable home of what ran.

### D2: Window check precedes the first Codex run, and a stop is a report

The 5-hour `used_percent` is read from the newest rollout under the scratch `CODEX_HOME` (or a one-token probe if none exists) before the first Codex run, and again after every Codex run.
Any refused or cut-off Codex run, or a reading of 70 or more, ends the Codex runs; the report then states the cell as unmeasured and names the reading.
Bound: the guard skips only the remaining Codex cells; the Opus runs do not depend on it and still run.

### D3: Opus solo runs through `lyx shuttle run`, interactive tmux, never `claude -p`

Each Opus run gets a fresh copy of the input's work directory, the same path re-pointing as `run.sh`, and the review file in `.bench/out/review.md` as the shuttle output file.
Wall time and tokens come from the shuttle outcome envelope and the strand's usage record.
Rationale: the project bills agents only as interactive tmux sessions.

### D4: Grading follows the earlier bench's method

One grader reads each finding against the Claude round-1 findings of the same input (`claude-findings.tsv` of the earlier bench) and, for an unmatched finding, against the cited artifact lines and snapshot code.
Labels are `shared`, `unique real`, `unique unchecked` and `false`, one row per finding in `data/match.tsv`.
Between the two new reviewers, a finding shared by both counts once as shared with the other, and each finding's match to the Claude fan is recorded separately.

### D5: Cap is hard

No fifth run, no repeat and no smoke test beyond the first real run.
A run whose harness failed before the model started (missing input, bad path) is rerun; a run that started and produced a bad review is not.

## Constraints

- `PATTERN.md` binds code; this task changes only `docs/research/` and `.scratch/`, so no PATTERN entry applies beyond `PATTERN-lyx-temp-root` (scratch lives under `.scratch/`, never the system temp directory).
- The Codex account is ChatGPT Plus: the plan's rate limits are the only cost signal, and `codex exec` runs with a scratch `CODEX_HOME` holding a linked `auth.json`.
- Markdown follows semantic line breaks.

## Auto-mode assumptions

- The weft checkout `loomyard-weft` beside the hub holds the plan artifacts at the earlier bench's commits.
- The earlier bench's `match.tsv`, `claude-findings.tsv` and `claude-usage.tsv` are the Claude fan reference and are not regenerated.
- The code commit of each input is the earlier bench's recorded one, not recomputed.

## Open risks

- Plus-plan limit: Astra cost 5 to 17 points of the 5-hour window per input in the earlier bench, so the two Astra runs may hit a window another session already filled.
- One run per cell and one grader: no variance, and the grading is not validated.
- Opus through an interactive session may read more of the snapshot than a `codex exec` run does; the reviewer prompt is identical, the harness is not.

## Acceptance criteria

- `docs/research/codex-astra-solo-bench-2026-10-25/report.md` exists and covers, for `P1` and `P4`, Astra and Opus solo: findings, shared, unique real, false, verdict, tokens, wall time and 5-hour window share.
- The report sets each result against the recorded Claude fan review for the same input.
- The data directory holds every review file, `match.tsv`, `usage.txt` and the harness scripts used.
- No more than four reviewer runs are recorded.
- A Codex cell that was refused or cut off is reported as unmeasured with the limit reading, not omitted.
- `lyx loom validate-description` passes on the change description.
