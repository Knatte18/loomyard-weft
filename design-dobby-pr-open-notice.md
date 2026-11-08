# PR-Review asks the parent orch for a review on its own, behind a hub and task setting

`dobby-pr-open-notice` · Task · feature, loom, burler, orch, config · running · Finalize

Imported from [issue #86](https://github.com/Knatte18/loomyard/issues/86).

## Seen

In hub `dop`, the operator finds the parent orch's review of a run's PR very useful, since the orch knows what the run needs.
Today the orch reviews a PR only when the operator tells the run's PR-Review reviewer to ask its parent orch for a review, as for PR 27 and PR 28, or asks the orch directly, as for PR 25 and PR 26.
Nothing is broken; the operator has to ask each time.

## Change

A setting, per hub and per task (a loom key with a `run_settings` override, beside the PR-Review perch's other keys), that makes the PR-Review step ask the parent orch for a review on its own when the run's PR is open.
The request names the slug, the PR URL and the branch.
With the setting on, the run waits at PR-Review until the orch answers satisfied or sends findings; findings are handled as the reviewer handles the orch's findings in the flow the operator triggers by hand today.
With the setting off, PR-Review runs as now.
The orch role gets the instruction for answering such a request.
