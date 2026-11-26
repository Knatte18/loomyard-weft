# A PR mode that registers the PR and approves it at once, with no agent under the block

`pr-auto-approve-mode` · Note · feature, landing, config

Imported from [issue #224](https://github.com/Knatte18/loomyard/issues/224).

## Seen (operator, 2026-11-24)

The operator often wants a pull request registered on GitHub without looking at it.
With a beam, say P1, and several tasks under it, the operator wants each task to land as a PR that is approved at once and merged into the beam.
When all of P1 is done, one PR goes from P1 into its parent (P0 or root), and it lists every member PR.
That lets the operator backtrack and see how the work got where it is.

Today the PR block has one setting: whether there is a PR or not.

## Idea

Add a third mode to that setting: PR on, auto-approved at once.

- The PR is made and shows up on GitHub, no agent is spawned under the PR-Review block, and the human need not look.
- The run goes on to Finalize as if the human had approved.
- A beam's own PR lists its member PRs.
  `Rewrite a beam's pull request description after each member landing` (`1cd11ee5e`) may already do part of this; read what it writes before adding more.
- This relies on the loops before the block having done every quality check they can, so the issue on taking the parent out of PR-Review belongs with it.
- Read first where the PR setting is read (`internal/landingshed`, `internal/pullrequest`) and how a run reaches Finalize today when a PR is approved.
