# A PR mode that registers the PR and approves it at once, with no agent under the block

`pr-auto-approve-mode` · Note · feature, landing, config

Imported from [issue #224](https://github.com/Knatte18/loomyard/issues/224).

## Seen (operator, 2026-11-24)

The operator often wants a pull request registered on GitHub without looking at it.
With a beam, say P1, and several tasks under it, the operator wants each task to land as a PR that is approved at once and merged into the beam.
When all of P1 is done, one PR goes from P1 into its parent (P0 or root), and it lists every member PR.
That lets the operator backtrack and see how the work got where it is.

## Decision (operator, 2026-11-26)

The PR block has three modes, and no LLM takes part in any of them:

- Manual: the PR is made and waits for the operator's own review and approval, as PR-Review is meant today.
- None: no PR; the row passes, as `pull_request` off does today.
- Trail (`pull_request: trail`, operator 2026-11-26): the PR is made and merged at once, with no agent and no human, so the beam's PR history can be backtracked.

How the three modes are configured is not decided yet.

## Idea

- Trail: the PR is made, no agent is spawned, and Finalize merges it at once, as it does today after the operator's `lym loom pr-continue`; merging a PR without an approval has long worked in lym.
- A beam's own PR lists its member PRs.
  `Rewrite a beam's pull request description after each member landing` (`1cd11ee5e`) may already do part of this; read what it writes before adding more.
- Manual waits for a human approval whatever the autonomy, which covers the note on autonomy and PR-Review.
- Read first where `beam.yaml` `pull_request` is read (`internal/landingshed`, `internal/pullrequest`) and how a run reaches Finalize today when a PR is approved.
