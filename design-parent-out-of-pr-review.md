# Take the parent out of PR-Review; the earlier reviewers must catch what it finds

`parent-out-of-pr-review` · Note · enhancement, landing, loom

Imported from [issue #223](https://github.com/Knatte18/loomyard/issues/223).

## Seen (operator, 2026-11-24)

Today the parent orch can be asked to review at PR-Review, and the operator has used that a lot.
The operator now judges it a bad idea, for two reasons.

PR-Review is the very last gate, and it belongs to the human.
When a run reaches it, everything the LLMs and LoomYard can do should already be done.

A parent is also the wrong reviewer.
Asking the parent, which is the orchestrator, to review a Discussion or code is work for the orchestrator, and it costs more than a cold-start agent.
The reviewers a run already has should catch what the parent catches.

## Decision (operator, 2026-11-26)

No agent reviews the pull request any more: when the PR is made, the agents are done.
What a parent finds when asked to review a PR must be found and fixed before the PR;
the checks that catch it are in the notes on reviewing the Describe text and on closing a task's scope at Code-Review.

## Idea

Take every agent out of PR-Review and add no seat in its place.

- Remove the parent's ask and re-ask from PR-Review in `internal/landingshed` and `internal/loomcli`.
- Remove PR-Review's own interactive session and its findings path: `pr_reviewer`, `lym loom pr-findings`, `pr_review_small_max`, `pr_review_ask_parent` and `pr_review_parent_timeout_min`.
- The row keeps only the mechanics the PR mode needs (see the note on the PR mode's three settings).
- Rely on the cold-start reviewers.
  The `review-code` fan already holds `code-scope`, which traces every decision of the record to a change in the diff and flags a decision no change builds, besides `code-correctness`, `code-affected-set` and `code-tests`.
  Read first which lenses the C# hubs run in `review-code` today, since the hub's own `lenses.yaml` and the C# layer can change the fan.
- Decide what a run does with a commit pushed to the PR branch by hand while it waits.
