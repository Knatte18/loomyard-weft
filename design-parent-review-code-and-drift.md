# Settings to route code review and drift judge rulings to the run's parent

`parent-review-code-and-drift` · Task · feature, loom, drift, burler

Depends on: [`review-frontmatter-hard-fail`](design-review-frontmatter-hard-fail.md)

Imported from [issue #153](https://github.com/Knatte18/loomyard/issues/153).

## What was seen

Only two reviews can be routed to the run's parent by setting: `pr_review_ask_parent` and `discussion_review_ask_parent` in `loom.yaml` or a task's `run_settings`.
The code review perch (Code-Burler-Review and Code-Burler-Fix) and the drift judge (Drift-Verify, Drift-Recheck) have no such key.
Their rulings reach the parent only when a finding is escalated as a decision call, or when an agent was told by hand to ask.
A parent that wants to see every ruling of a darn run, which has no Discussion, cannot get it by configuration.

## Wish

Add parent-review keys for the code review perch and the drift judge, parallel to `discussion_review_ask_parent`:

- `code_review_ask_parent`: each code review round mails the parent a `[code-review request]` naming the review and fixer report, and waits for `lym loom parent-review answer` (satisfied or a findings file) before the Bouncer judges the round, with a timeout key as `discussion_review_parent_timeout_min` has.
- `drift_judge_ask_parent`: the drift judge mails the parent its proposed rulings, every cell or a summary by case with the flagged cells named, and waits for the parent's agreement or changed rulings before it writes the judgement, with a timeout.

Both settable in `loom.yaml` (or `drift.yaml`) and in a task's `run_settings`.

## Workaround

Tell the run's agents by creel to ask the parent before they finish a round or write a judgement.


## Issue comment, 2026-11-19

A related need: a run's agents (drift judge, Drift-Expect, perch fixer) put their questions to the operator in their own pane. A setting that sends every question an agent would put to the operator to the run's parent by creel instead, with the parent's answer delivered back, would let the parent settle them without the operator relaying. Today the parent sees such a question only when the operator relays it or tells the agent to ask the parent.

## Issue comment, 2026-11-19

A second wish in the same direction: let the parent subscribe to a run's state, so lym mails it when the run reaches a named producer or state, for example `lym loom notify <slug> --on Drift-Verify,Drift-Expect,Code-Burler-Fix`, or a `run_settings` key such as `notify_parent_on: [Drift-Verify]`. The parent can then brief each agent of those rows as it starts, such as telling the drift judge to put its questions to the parent, without a polling watch on `lym loom status --watch`.