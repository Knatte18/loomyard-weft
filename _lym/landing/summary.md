# Bench the tuned forked Codex Astra fan on P4 at medium effort

Adds `docs/research/codex-astra-tuned-p4-med-bench-2026-10-27/`, a report and its data for one run of the tuned-prompt bench's setup with the effort changed from high to medium.
No production code, `PATTERN.md` or board entry changes.

## Result

One `codex exec` handler of `gpt-6-astra` at medium effort spawned three lens forks with `fork_turns="all"` over the `P4` plan, with the Claude reviewers' context, `AGENTS.md` and the tuned reading block unchanged.
It returned `BLOCKING` with 6 findings: 3 shared with the Claude fan, 3 `unique-real`, none `false`.
It found 3 of the fan's 11 findings and 2 of its 3 `BLOCKING` ones (`adoptcli` missing from `PATTERN-cli-cobra`'s background, and the one-second production wait in `TestSetOwners`); the `retryWeftPush` owner guard was missed.
The high-effort run found 4 of 11 and the same 2 of 3.
The run took 492 s and 236k fresh input tokens, against 950 s and 631k at high effort.

## Design decisions

- The harness is the tuned-prompt bench's with the path root and the effort changed, plus a new `evidence.sh` that prints the rollout evidence; `tuned-block.md` and the context files hash equal to that bench's.
- The rollouts passed both checks the task names: the forks started with `fork_turns="all"`, overlapped for about two minutes and inherited the handler's context, and `AGENTS.md` loaded in every rollout.
- The fan's finding texts are not in the repository, so the grading against the fan rests on the earlier benches' notes and severities; the report says so.
- The weekly window read 18 percent at the first request and 27 at the end; sibling benches may share the account, so the report does not attribute the points to this run.

## Testing

Nothing built and no test applies: the commit adds only files under `docs/research/`, which no build or test reads.
