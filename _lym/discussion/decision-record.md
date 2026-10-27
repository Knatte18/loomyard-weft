## Goal

Bench Codex Astra's forked lens fan on plan `P1` with reading block v2 and a raised fork tool-call cap, and report whether it finds the Claude fan's findings the earlier blocks missed and whether one high run or the union of two medium runs is the better buy.

## Scope

- One input, `P1` (`gate-runs-once`), rebuilt from the weft and code commits in `docs/research/codex-astra-tuned-p1-high-bench-2026-10-27/data/inputs.txt`.
- Three `codex exec` handler runs of `gpt-6-astra`, each spawning three lens forks with `fork_turns="all"`: one high and two medium, identical but for effort.
- Output is `docs/research/codex-v2-p1-bench-<date>/report.md` with `data/` (harness, prompts, block v2, reviews, rollout evidence, usage, `match.tsv`).
- No Claude run, no other repeat, no product code change.

## Decisions

- Reuse the `codex-astra-tuned-p1-high-bench-2026-10-27` harness with the root and bench directory changed to this worktree and bench directory.
  Rationale: the board entry asks for the tuned-prompt bench's harness and context unchanged.
- Write block v2 word for word to `data/prompts/block-v2.md`, in place of block v1, in the handler prompt and the fork boilerplate.
  Rationale: block v2 is the one variable under test.
- Change the fork boilerplate's cap sentence from `at most 6 tool calls` to `at most 20 tool calls` and nothing else.
  Rationale: the entry fixes this as the only boilerplate change; the build script asserts the substitution applied.
- Run the three handlers in parallel, each in its own copy of the input and its own `CODEX_HOME` sessions directory scan by marker file.
  Rationale: the union's wall time counts the medium runs as parallel, and the sibling benches already share the account.
- Grade with the earlier benches' labels (`shared`, `unique-real`, `false`) against the Claude fan reference of `P1` in the weft's `round-1-review.md`, into `data/match.tsv`.
  Rationale: rows must compare with the earlier `P1` rows.
- Stop and report what was done if a window reading passes 70 percent, or a rollout shows forks not started, not concurrent, not inheriting context or `AGENTS.md` unloaded.
  Rationale: the entry makes both conditions stop conditions.

## Constraints

- Effort is `high` or `medium`, never `xhigh` or above.
- Scratch, auth link and run cells live under `.scratch/` in this worktree, which is untracked.
- No tracked file outside `docs/research/codex-v2-p1-bench-<date>/` changes.
- The report records the wall time, token counts, per-fork command counts and window readings, and says that sibling benches ran on the same account at the same time.

## Auto-mode assumptions

- No operator is present; best-judgment calls stand and are named in the change description.
- The grader is this Claude session, as in the earlier benches.
- The date in the output directory is the day of the run.

## Open risks

- One input and one run per cell: the high-versus-medium comparison and the medium-to-medium difference cannot separate effort from chance.
- A fork's prompt is encrypted in the rollout, so that a fork's text held block v2 rests on the handler following the cluster rules.
- Window readings include the sibling benches' draw.

## Acceptance criteria

- The report holds the summary table with the high run, each medium run alone and the union of the two medium runs, each with findings, Claude fan findings found, `BLOCKING` found, unique real, false, fresh input tokens, window points and wall time.
- The report says per Claude fan `BLOCKING` finding on `P1` which run found it, how far the two medium runs differ, and lists every earlier Astra and Claude row for `P1` beside the new ones.
- `data/rollout-evidence.txt` shows per run that forks started, ran concurrently and inherited context, and that `AGENTS.md` loaded.
- `data/prompts/block-v2.md` equals the entry's block v2 byte for byte, and the run prompts differ from the tuned bench's only in the block and the cap number.
