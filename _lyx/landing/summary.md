# Bench Codex as a Burler reviewer against Claude reviews

Adds a research report that measures whether Codex reaches the Claude reviewers' quality as a Burler reviewer of discussions and plans, and at which model and effort.
It is the evidence asked for before note `codex-engine` is built.
Nothing under `internal/`, `cmd/`, `contracts/` or `PATTERN.md` changes; every file is under `docs/research/codex-review-bench-2026-10-25/`.

## What is added

- `report.md`: conclusion, summary table of model and effort against shared, codex-only real, false and claude-only counts with tokens, wall time and plan-window use, the method, the `codex exec` and billing findings, and the limits of the comparison.
- `data/`: the raw Codex review files, `usage.txt` (per-run tokens, wall time, rate-limit readings), `match.tsv` (every Codex finding matched to Claude's findings of the same round), `claude-findings.tsv` and `claude-usage.tsv` (the Claude reference), `inputs.txt` (the ten inputs with weft and code commits), `operator-codex-home.txt`, and the shell harness under `data/harness/`.

The conclusion on this sample: Codex does not reach the Claude fan's coverage.
`gpt-6.1-sol` at medium effort is the only setting worth a second look, as an extra non-gating reviewer.
The bench stopped early on the plan's 5-hour limit, and the report names the unmeasured cells.

## Design decisions

- Inputs are round-1 discussion and plan reviews of landed `loomyard-weft` runs, each paired with the artifact and with the code at the last parent commit before the Claude round started.
- The prompt is the production Burler reviewer chain, solo, so Codex is measured against a Claude lens fan; the report states this caveat.
- Codex runs with `CODEX_HOME` under `.scratch/` linking `~/.codex/auth.json`, work directory under `.scratch/`, `-s workspace-write`.
  The sandbox confines writes, not reads; the report says so.
- Grading is by one Claude session against the artifact and code, with the match table committed as the audit trail.
- The bench is a measurement record; a later run supersedes it with a new dated directory.

## Check by hand

Not run or not settled in this change:

- `data/usage.txt` carries the window percentages per run, and the report's billing figures follow from them.
- The report's evidence that `~/.codex/config.toml` is unchanged rests on file times, not a hash taken before the bench; the unexplained `~/.codex` directory mtime at 12:15 is still unexplained.
- Every codex-only finding in `data/match.tsv` is graded against the artifact and code, and the fixer report and bouncer verdict were read for them.
- `data/harness/grid.sh` stops at 70 percent used in either plan window; the bench itself ran past that.
- The `BLOCKING` counts in the report can be recomputed from `data/claude-findings.tsv` and `data/match.tsv`.
- No run read a path under the weft checkout, since workspace-write does not confine reads.
- `grid.sh` reruns refused cells instead of skipping them.
- The conclusion's count of Sol findings checked against code matches the findings list.
- `fill()` in `data/harness/build.sh` quotes its replacement, so an ampersand in a substituted value does not corrupt the prompt.
- `grid.sh` reads the rollout of the run it just made, not the previous run's.
- The harness scripts run as committed, and the report's `harness/` description says which steps were done by hand.
- `report.md` breaks lines at independent-clause boundaries.
