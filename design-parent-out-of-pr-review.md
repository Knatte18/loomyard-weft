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

## Idea

Take the parent out of PR-Review and add no parent seat in its place.

- Remove the parent's ask and re-ask from PR-Review in `internal/landingshed` and `internal/loomcli`, leaving the human's review there.
- Rely on the cold-start reviewers.
  The `review-code` fan already holds `code-scope`, which traces every decision of the record to a change in the diff and flags a decision no change builds, besides `code-correctness`, `code-affected-set` and `code-tests`.
  Read first which lenses the C# hubs run in `review-code` today, since the hub's own `lenses.yaml` and the C# layer can change the fan.
- What the parent found at PR-Review that the lenses do not cover is in the issue on reviewing the Describe text and catching the rest earlier: the PR text, numeric claims that need the report open, bugs that need a measurement, and scope the Sketch never held.
- Decide what a run does with a parent ruling that arrives late: it never sends the run back from the human's gate.
