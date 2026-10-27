# Bench the forked Codex Astra fan with the Claude reviewers context on P4

Adds `docs/research/codex-astra-context-parity-bench-2026-10-27/` with its `report.md` and `data/`.
No Go code, PATTERN entry or module doc changes.

## What the bench shows

One `codex exec` handler at `gpt-6-astra`, high effort, spawned three lens forks with `fork_turns="all"` on plan `P4`, with the context the Claude fan's round 1 had: the `RoleReviewFix` PATTERN directive inlined in the explore step, and the snapshot's `CLAUDE.md` loaded as `AGENTS.md`.
The handler found 2 of the Claude fan's 11 findings and none of its 3 `BLOCKING` ones, against 1 of 11 and none for the same fan without the context.
The gained finding is the fan's `F6`, card 17's shortened `PATTERN-owner-machine` line.
The fan's `F2`, `adoptcli` missing from `PATTERN-cli-cobra`'s background, was not found although the handler read that background.
The run took 980 s and 324k fresh input tokens.
The rollouts confirm the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` was loaded.

## Design decisions

- The harness is the forked fan bench's, copied with its paths moved to this worktree; `build.sh` alone changes, to fill `pattern_directive` from the snapshot's own stencils and copy `CLAUDE.md` to `AGENTS.md`.
- `evidence.sh` is new: it prints the rollout evidence the forked fan bench gathered by hand.
- The handler's `-o` last message was empty, so its review is the file it wrote; each fork's review is the last assistant message of its rollout.
- The fork finding the handler dropped is graded in `data/match.tsv` as `unique-real` and arguable, as the card's wording leaves the handler's rejecting reading open.

## Limits

One input, one run, one grader; the weekly window moved 9 to 17 percent with a sibling bench running on the same account, so the window points are not this run's alone.
