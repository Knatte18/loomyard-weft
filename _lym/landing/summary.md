# PR-Review asks the parent orch for a review behind pr_review_ask_parent

A new loom key, `pr_review_ask_parent`, makes the PR-Review row ask the run's parent orch to review the pull request on its own once the pull request is open.
Before this, the orch reviewed a run's pull request only when the operator asked for it each time.
The key is off by default, so PR-Review runs as before unless a hub or a task turns it on.

## What changes

- `loom.yaml` gains `pr_review_ask_parent: false`.
  It is open to the local layer, so a task's `run_settings` turns it on or off for one run.
- The PR-Review row asks once it has its interactive session and before it waits.
  It sends one creel line, signed by the hub's `LOOM` program, to the nearest live orch above the pair.
  The line opens `[pr-review request]` and names the slug, the pull request URL, the task and parent branches, and the PR-Review session's strand.
  It tells the orch to answer that strand with `lym creel post <strand> "<answer>"`, saying it is satisfied or giving its findings.
- The pull request record gains `parent_review_asked`, so a re-entry of the row does not ask again.
  A review that starts afresh writes a new record, so the orch is asked again.
- An unreadable setting, a failed post, or no orch recorded to take the line is logged.
  The record is left unasked, so the row's next entry asks again.
- The PR-Review stencil gets a "Your parent's review" section.
  The agent fixes the parent's findings the way it handles the operator's requests, commits, tells the operator what changed, and asks the parent to look again until it is satisfied.
  When the setting is off, the agent asks its parent only when the operator tells it to, as in today's manual flow.
- The orch role gets a "Reviewing a pull request" section.
  The orch hands the diff read to a one-shot fork and judges the result against what it knows the run needs.
  It answers the session the request names and never approves, merges, closes or comments on the pull request.

## Design decisions

- The key is in `loom.yaml`, not `landing.yaml`, beside `pr_reviewer`.
  `landing.yaml` is hub-wide and closed to the local layer, so a task's `run_settings` could not override a key there.
- The row sends the request itself, not the reviewer agent, so the ask does not depend on the agent's judgment.
  The orch's answer goes to the reviewer session, which already handles findings.
- The row's endings do not wait on the orch's answer.
  The run already waits at PR-Review until the operator ends the review or approves or merges the pull request, so it stays there while the orch reviews.
  A gate that blocked the operator's own ending until the orch answered could leave a run stuck when no orch answers.
  The operator stays the one who ends the review.

## Testing

- `TestPRReview_Session` gains rows for each case:
  an ask with the setting on, checking every field the request names and the record flag;
  no ask when the record already holds one;
  no ask when the setting is off;
  no ask when the setting cannot be read;
  and an unasked record after a failed post, or when no orch is recorded to take the line.
- The loom config test gains a row that loads `pr_review_ask_parent: true` and checks that it is read.
- The `landingDeps` drift test covers the two new `PullRequestDeps` seams by reflection.
