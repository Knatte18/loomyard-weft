## Goal

Measure how Codex Astra's forked lens fan, given the Claude reviewers' context and the tuned reading block, reviews the `P1` plan (`gate-runs-once`) at medium effort, and report it beside the tuned-prompt bench's `P4` row and every earlier Astra and Claude row for `P1`.

## Scope

- One new directory, `docs/research/codex-astra-tuned-p1-med-bench-2026-10-27/`, with `report.md` and a `data/` holding the harness, prompts, reviews, rollout evidence, usage figures and `match.tsv`.
- One input, `P1`, rebuilt from the weft commit and code commit in `docs/research/codex-astra-solo-bench-2026-10-25/data/inputs.txt`.
- One `codex exec` handler at `gpt-6-astra` medium effort that spawns 3 lens forks (`affected-set`, `guard-reach`, `way-forward`, the lenses the Claude fan's round 1 on `P1` recorded) with `fork_turns="all"`; no repeat, no Claude run.
- Grading of the handler's findings, and of any fork finding the handler dropped, against the Claude fan reference of `P1` (12 findings, 2 `BLOCKING`), as the earlier benches graded.
- No change to production code, `PATTERN.md` or any module doc.

## Decisions

### Reuse the tuned-prompt bench's harness with only paths changed

Decision: copy `build.sh`, `build-fan.sh`, `build-forkfan.sh`, `run-forkfan.sh`, `setup-codex-home.sh`, `windows.sh` and `dump.sh` and `tuned-block.md` from `docs/research/codex-astra-tuned-prompt-bench-2026-10-27/data/` into the new `data/`, change `ROOT` and the block path to this worktree, and change `effort=high` to `effort=medium` in the runner.
Rationale: the task keeps every condition of that bench and changes only the input and the effort.
The block, the inlined PATTERN overview and `AGENTS.md` are produced by the same scripts, so they are unchanged word for word.

### Run once, stop on the window cap

Decision: run the handler once; read the weekly and 5-hour `used_percent` of the rollout's `token_count` events before the call and after it, and report the run as stopped rather than continue if either passes 70 percent.
Rationale: the cap names one run and a 70 percent stop.
Bound: the run cannot start a second call; the check reads the account's window, shared with sibling benches, which the report says beside the figures.

### Stop if the fork evidence fails

Decision: confirm from the rollouts that three `spawn_agent` calls carry `fork_turns="all"`, that the forks' time ranges overlap, that each fork's `forked_from_id` is the handler's session with a large first request, and that `AGENTS.md` loaded in each rollout; when any fails, report that and grade nothing.
Rationale: the task makes this a stop condition, since a fan without live, context-inheriting forks is not the setting under test.

### Grade as the earlier benches did

Decision: one pass grading each handler finding `shared`, `unique-real` or `false` against the fan's 12 findings in `docs/research/codex-review-bench-2026-10-25/data/claude-findings.tsv` and the round-1 review in the weft, checking an unmatched finding against the cited card and the snapshot's code; record it in `data/match.tsv` with the earlier column layout.
Rationale: the report is compared to the earlier rows, so the labels and the matching rule stay the same.

### Report layout follows the tuned-prompt bench's report

Decision: `report.md` carries Conclusion, Summary table, Per call, Fork evidence, Context and prompt, the handler's findings, Method, Deviations and limits, and Data, with a per-finding statement for each of the fan's two `BLOCKING` findings and the earlier `P1` rows (cold-call comparison rows are those recorded for `P1` only).
Rationale: a reader compares the two reports section by section.

## Constraints

- `gpt-6-astra` at `medium` for the handler and its forks; never `xhigh` or above.
- Scratch work under `.scratch/` in this worktree; the scratch `CODEX_HOME` holds only a link to the operator's `auth.json`.
- Nothing is pushed; the run's commit lands the `docs/research/` directory only.
- `PATTERN-lym-temp-root` and the other PATTERN entries bind code; this change adds shell scripts and markdown under `docs/research/` and no Go.
- Markdown follows semantic line breaks, one sentence per line.

## Auto-mode assumptions

- The Claude fan's `P1` round is the reference as recorded in `claude-findings.tsv` and the weft's `round-1-review.md`; it is not rerun.
- The `P1` rows of the earlier benches that exist are the solo bench's Astra and Opus rows and the review bench's Astra row; no earlier fan or forked-fan run of Astra exists for `P1`.
- Sibling benches (`codex-astra-tuned-p1-high-bench`, `codex-astra-tuned-p4-med-bench`) may draw on the same account while this one runs.

## Open risks

- One input, one run and one grader cannot separate the effect of medium effort from run-to-run variance.
- The rate-limit window may rise from sibling runs alone; a stop at 70 percent would leave the bench with no result.
- `P1` is a 33-card plan, larger than `P4`, so the handler's first reading may be long and the forks' inherited context larger.

## Acceptance criteria

- `docs/research/codex-astra-tuned-p1-med-bench-2026-10-27/report.md` exists with the sections above and a `data/` holding every file the report names.
- The report states the wall time, the handler's and each fork's fresh input and output tokens, and the windows before and after, with the note on sibling benches.
- The report states, for each of the Claude fan's two `BLOCKING` findings on `P1`, whether this run found it.
- Every handler finding, and every dropped fork finding, has a row in `data/match.tsv` with a grade.
- The summary table puts this run beside the tuned-prompt bench's `P4` row and every earlier Astra and Claude row for `P1`.
- The rollouts show three `spawn_agent` calls with `fork_turns="all"`, overlapping fork runs, inherited context and a loaded `AGENTS.md`; or the report is a stop report naming the failure.
