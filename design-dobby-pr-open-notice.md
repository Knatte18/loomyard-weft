# Dobby: configurable notice to the orch when a run opens its PR

`dobby-pr-open-notice` · Note · feature, dobby, orch, config

Imported from [issue #86](https://github.com/Knatte18/loomyard/issues/86).

## Seen

In hub `dop`, the operator wants the orch to review every run's PR and send findings to the run until satisfied.
The orch role forbids a Monitor or polling on runs, so the orch learns of a PR only when the run's PR-Review reviewer strand chooses to mail it.
That works today (the pr-reviewer mailed the orch for PR 26 and PR 27), but it depends on the PR-Review perch's own prompt, and the operator cannot see or set it as a guarantee.

## Expected

A setting, per hub and per task (for example in `batten.yaml`, `loom.yaml` or the task's `run_settings`), that makes dobby send the parent orch a `[dobby notice]` when a run opens its PR, naming the slug, PR URL and branch.
With it on, the run waits at PR-Review until the orch replies "satisfied" or sends findings, as the pr-reviewer flow does now.

## Workaround

None needed in practice: the orch relies on the pr-reviewer strand's mail.


## Issue comment, 2026-11-08

Correction to Seen: the orch got the pr-reviewer's mail only for PR 27. For PR 25 and PR 26 no mail reached the orch; the operator had to ask the orch to review them. So the orch is not reliably told when a run opens its PR, which is the gap this issue asks to close.

## Issue comment, 2026-11-08

Correction to my previous comment: nothing is broken. By design the operator tells the PR-Review reviewer to ask its parent orch for a review; that is how PR 27 and PR 28 reached the orch, and PR 25 and PR 26 did not because the operator asked the orch directly instead. The operator finds the parent's review very useful, since the orch knows what the run needs. This issue is the feature request only: a setting, per hub and per task, that makes the PR-Review step ask the parent orch for a review on its own, so the operator does not have to ask each time.