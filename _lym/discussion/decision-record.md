## Goal

Bench one Codex Astra lens fan whose three lens forks are real forks, inheriting the handler's context, at high effort on plan `P4`.
Report whether it finds what the Claude fan found, beside the earlier Astra fan, solo Astra, solo Opus and Claude fan rows.

## Scope

- One input: `P4`, the `machine-handover` plan, rebuilt from the earlier bench's `data/inputs.txt` with its harness, prompts and grading.
- One model and effort: `gpt-6-astra` at high effort, for the handler and its forks.
- One run: a single `codex exec` handler that does the explore step once, then spawns three lens forks in parallel through `spawn_agent` with `fork_turns="all"`.
- No Claude run; the Claude fan reference is the one the earlier benches recorded.
- Output: `docs/research/codex-astra-fork-fan-bench-<date>/report.md` with its `data/`, landed as the run's commit.
- No Go code and no PATTERN change; the bench is shell scripts, the reviewers' output and a hand grading.

## Decisions

- Decision: the harness is copied from the earlier bench's `data/harness/` into the new `data/harness/` and adapted, not shared by reference.
  Rationale: each report dir stays self-contained and reproducible.
- Decision: the handler prompt is the round-1 reviewer template with the fan cluster rules and the spawn paragraph kept, and the fork prompt keeps its inherited-context sentence and its tool-call cap.
  Rationale: the question is whether real forks match Claude's, so the fork boilerplate stays Claude's.
- Decision: the multi-agent feature is chosen by a probe of `codex exec` before the real run; the feature enabled, `multi_agent` or `multi_agent_v2`, is recorded in the report.
  Rationale: the task lets either enable `spawn_agent` with `fork_turns`, and the installed CLI decides which does.
- Decision: the rollout is checked before any grading: three forks started, ran concurrently and received the handler's context.
  If not, the run stops and the report states what the rollout shows; no fallback to cold calls runs.
  Bound: this check can only stop the bench, it never substitutes another run shape.
- Decision: grading follows the earlier bench: the handler's findings and any dropped fork finding against the Claude fan's `P4` findings, `shared`, `unique-real` or `false`, in `data/match.tsv`.
- Decision: the rate-limit window is read after the run and before any further call; a window past 70 percent stops the bench and the report says what was done.

## Constraints

- The task cap: one run, no repeat, no Claude run, one model and effort.
- Scratch files live under `.scratch/` and are not committed.
- The operator's `~/.codex/auth.json` is linked into a scratch `CODEX_HOME`; the operator's `config.toml` is not edited.
- Markdown uses semantic line breaks.

## Auto-mode assumptions

- Best-judgment calls replace operator dialogs; a probe of the multi-agent feature costs at most one short extra `codex exec`.
- The report date is the date of the run.

## Open risks

- `spawn_agent` with `fork_turns` may be absent or behave differently in the installed CLI; the rollout check then stops the bench.
- One input, one run and one grader measure no variance.
- Forks may not run concurrently if the handler spawns them one by one.

## Acceptance criteria

- `report.md` states the conclusion, the summary table with the four earlier rows and the new row, per-call tokens and wall time, the features enabled, and the deviations.
- The rollout evidence for three forks, their concurrency and the inherited context is cited in the report, or the report states the stop.
- `data/match.tsv` grades every handler finding and every dropped fork finding.
- `data/` holds the harness, prompts, reviews, `usage.txt` and `inputs.txt`.
