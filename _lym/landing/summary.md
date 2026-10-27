# Bench a Codex Astra forked fan with a tuned reviewer prompt on P4

Adds `docs/research/codex-astra-tuned-prompt-bench-2026-10-27/` with its `report.md` and `data/`: one forked Codex Astra fan on the `machine-handover` plan `P4`, run with the Claude reviewers' context and one block of reading directions.
No Go code, stencil or `PATTERN.md` changes.

## Result

At `gpt-6-astra` high effort, the handler and its three lens forks found 4 of the Claude fan's 11 findings and 2 of its 3 `BLOCKING` ones, where the forked fan bench without the context and block found 1 and none.
The two found are `adoptcli` missing from `PATTERN-cli-cobra`'s background, and the one-second production wait in `TestSetOwners`.
The owner guard on `retryWeftPush` reading origin's Bolt outside `ReconcileOrigin` was not found.
The run took 950 s and 631k fresh input tokens against 737 s and 299k.
Rollouts confirm three concurrent forks that inherited the handler's context, and an `AGENTS.md` load in the handler and each fork.
The report grades all 10 handler findings and the one dropped fork finding in `data/match.tsv`: 4 `shared`, 7 `unique-real`, none `false`.

## Design decisions

- The harness is the forked fan bench's, with the PATTERN directive added to the explore instruction and `CLAUDE.md` copied to `AGENTS.md`; the directive is built from the code commit's `pattern-directive-review-fix.md`, `PATTERN.md` and the decision record's declared change, as `pattern.Directive` renders it for the reviewer.
  The decision record said the changes marker stays empty; the run includes the declared `PATTERN-owner-machine` change because production renders it for this plan, so the context matches the Claude round's.
- The tuned block sits in the handler prompt and, as one line, in the fork boilerplate, since the cluster rules allow a fork nothing beyond the boilerplate and its lens; the 6-command cap per fork stays.
- The sibling task `codex-astra-context-parity-bench` had written no files when this run was built, so the context files were built from the two board entries' shared description; `data/context-hashes.txt` lets the two be compared.
- The three changes against the forked fan bench, context, `AGENTS.md` and block, were made together as the task specifies, so the report states that it cannot attribute the gain to the block.

## Checks

- `data/` holds the harness, prompts, reviews, usage and rollout evidence; `.scratch/` stays uncommitted.
- Docs-only commit: no module, CLI behavior or PATTERN invariant changed.
