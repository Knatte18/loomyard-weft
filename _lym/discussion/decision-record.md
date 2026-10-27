## Goal

Measure whether three of the gap study's lens and boilerplate texts raise the real findings per weekly-window point of the forked Codex Astra fan on plan `P2`, against the two medium runs of `codex-v2-p2-bench`.
The deliverable is `docs/research/codex-p2-gap-texts-bench-2026-10-27/report.md` with its `data/`.

## Scope

- Reuse `docs/research/codex-v2-p2-bench-2026-10-27/data/` harness, inputs, context and prompts, changed only by the three texts below, each copied word for word from `docs/research/claude-review-gaps-2026-10-27/report.md`.
- Proposal 1: the replacement last sentences of lens `guard-reach`, and its reading bullet added to the fork-reading block.
- Proposal 2: the added `guard-reach` sentence, and "one hop beyond it" changed to "beyond it along the input's path" in the fork boilerplate.
- Proposal 4: the added `way-forward` sentences.
- Exactly 2 runs, `gpt-6-astra` at `medium`, forked fan (`spawn_agent`, `fork_turns="all"`, three lens forks), `P2` only, fork cap 20, run one after the other.
- Nothing under `internal/` or `contracts/` changes; no PATTERN change.

## Decisions

- The bench harness is copied into the new bench's `data/harness/` with its `ROOT` changed to this worktree, so the committed scripts reproduce the bench.
  Rationale: the earlier harness names a worktree that no longer exists.
- The `guard-reach` lens text is the original with Proposal 1's three sentences, then Proposal 2's added sentence, then Proposal 1's `Your list`.
  Rationale: Proposal 1 replaces the last two sentences including the old `Your list`, and Proposal 2 adds before `Your list`; this order satisfies both.
- Proposal 1's reading bullet goes into `block-v2.md` after the bullet on the functions a card changes and their callers, the closest match to the gap study's "after the bullet on callers and callees".
  Rationale: block v2 is the fork-reading block of the tuned benches.
- The 20-call fork cap stays, and block v2's own "one hop beyond the cards" bullet stays unchanged.
  Rationale: Proposal 2 names the fork boilerplate's phrase only.
- The two runs run in sequence, each with a scratch `CODEX_HOME`; the weekly `used_percent` is read before and after each run from the rollouts' `token_count` events, the "before" of the first run from the newest earlier rollout under `~/.codex/sessions`.
  Rationale: the task requires each run's window points to be its own.
- Grading follows the earlier bench: one Claude session, each handler finding against the fan's round-1 findings of `P2` and, if unmatched, against the cited card lines and snapshot code.
  Rationale: same grader and rubric keep the rows comparable.

## Constraints

- Exactly 2 Codex runs: no high run, no other model, no other plan, no repeat.
- Codex weekly budget is nearly used up: a failed run is not repeated beyond the cap, and a refusal by the usage limit is reported as such.
- `CLAUDE.md`: semantic line breaks in every `.md` file; no `update-plugins.sh`; no push.
- Bench scratch lives under `.scratch/`, not committed.

## Auto-mode assumptions

- No operator is present; a pause between the runs is not needed beyond sequencing them.
- The operator's `~/.codex/auth.json` may be linked into the scratch `CODEX_HOME`, as the earlier bench did.

## Open risks

- A usage-limit refusal can end a run with no result; the report then states it and the cap still holds.
- Two medium runs have a run-to-run spread of one or two findings, so a difference that small is no result.
- The "before" window reading of the first run may predate other use of the account.

## Acceptance criteria

- `report.md` holds the per-run measures the board entry lists: findings, real, `BLOCKING` real, false, the fan's `BLOCKING` findings found, commands per fork, `way-forward` fork findings, fresh input tokens, window points, wall time.
- `report.md` states real findings and real `BLOCKING` findings per window point against the two medium runs of `codex-v2-p2-bench`, and that a difference of one or two findings is inside the run-to-run spread.
- `data/` holds the three texts' diff against the earlier prompts, the harness, the reviews, usage and rollout evidence, and the grading.
- Exactly 2 rollout handlers exist for the bench.
