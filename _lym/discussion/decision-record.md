## Goal

Measure how Codex Astra's forked lens fan does on `P4` at medium effort with the Claude reviewers' context and the tuned reading block, and report it beside the high-effort tuned bench and every earlier `P4` row.

## Scope

- One new report, `docs/research/codex-astra-tuned-p4-med-bench-<date>/report.md`, with its `data/`, landed as the run's commit.
- One input: `P4`, the `machine-handover` plan, from the weft commit and code commit in `docs/research/codex-astra-solo-bench-2026-10-25/data/inputs.txt`.
- One run: one `codex exec` handler of `gpt-6-astra` at medium effort spawning 3 lens forks with `fork_turns="all"`; no repeat, no Claude run.
- The harness, context and `tuned-block.md` of `docs/research/codex-astra-tuned-prompt-bench-2026-10-27/data/` are reused; only the effort and the path root change.
- No production code, no `PATTERN.md` change, no board change.

## Decisions

- The harness scripts are copied into the new bench's `data/harness/` with `ROOT` pointing at this worktree and the effort set to `medium`; nothing else in them changes.
  Rationale: the cap changes the effort only, so any other difference would confound the comparison with the high-effort bench.
- `tuned-block.md` is copied word for word, and a hash in `data/context-hashes.txt` shows it equals the tuned bench's.
  Rationale: the comparison holds only if the reading directions are identical.
- The lenses are `affected-set`, `guard-reach` and `way-forward`, the three the Claude fan's round 1 on `P4` used, taken from the snapshot's `internal/lenses/template.yaml`.
  Rationale: the board entry names the Claude fan's lenses, and the harness reads them from the template.
- The grading follows the earlier benches: each handler finding and each dropped fork finding is `shared`, `unique-real` or `false` against the fan reference in `docs/research/codex-review-bench-2026-10-25/data/claude-findings.tsv` and the fan's recorded findings, with unmatched findings checked against the cited card lines and the snapshot's code.
  Rationale: rows only compare across benches when graded the same way.
- The run is judged valid only if the rollouts show the forks started, ran concurrently and inherited the handler's context, and `AGENTS.md` loaded; a failure ends the task with a stop report instead of a graded result.
  Bound: this check cannot read the encrypted fork prompts, so it rests on the `spawn_agent` calls, the overlap of the forks' times, `forked_from_id` and the first request sizes.
- The run stops and reports what was done if a rate-limit window passes 70 percent; the window is read from the rollouts' `token_count` events, and siblings on the same account are named beside the figures.
  Bound: the stop applies between steps, so a single call that crosses 70 percent is not interrupted.

## Constraints

- `gpt-6-astra` at `medium` for the handler and its forks; never `xhigh` or above.
- No Python; scripts are shell, as the earlier bench's harness is.
- Scratch files live under `.scratch/`, which is never committed.
- Markdown uses semantic line breaks; table cells stay on one line.
- PATTERN constraints bind only code, and this task adds none.

## Auto-mode assumptions

- The operator's `~/.codex/auth.json` is linked into a scratch `CODEX_HOME` as in the earlier benches, and its `config.toml` does not apply.
- The earlier bench's grading judgments for findings that recur on `P4` (the skip-variable bypass, the `Lost` rule, the bundled public symbols) are reused where the new run states the same defect.

## Open risks

- One input, one run and one grader cannot show variance or an effort effect; the report says so.
- A sibling bench on the same account may move the window figures; the report says so beside them.
- The grader is a Claude session and may favour its own family's phrasing.

## Acceptance criteria

- `report.md` holds a conclusion, a summary table with this run beside the high-effort tuned bench row and every earlier `P4` Astra and Claude row, a per-call table, the fork evidence, the findings with grades, the method, and the deviations and limits.
- The report states, for each of the Claude fan's three `BLOCKING` findings on `P4`, whether this run found it.
- The report records wall time, the handler's and each fork's fresh input and output tokens, and the windows with the sibling-bench caveat.
- The report's fork evidence shows the three `spawn_agent` calls with `fork_turns="all"`, overlapping fork times, the forks' `forked_from_id`, and the `AGENTS.md` hash equal to the snapshot's `CLAUDE.md`.
- `data/` holds `inputs.txt`, `usage.txt`, `rollout-evidence.txt`, `context-hashes.txt`, `match.tsv`, `reviews/P4/`, `prompts/` and `harness/`.
